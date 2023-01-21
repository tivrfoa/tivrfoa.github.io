---
layout: post
title:  "Rust for Linux 2023-01-20"
date:   2023-01-20 07:30:00 -0300
categories: Rust Linux kernel
---

<!--
<div style="background-color: #a2b9bc; padding: 15px; border-radius: 10px; margin-bottom: 10px">
Me: Doesn't the second version creates new String objects on every call, hence causing an impact in GC?
</div>
-->

### Use sysfs api instead of /proc


```txt
Greg KH <gregkh@linuxfoundation.org>
	
Sat, Jan 7, 7:54 AM
	
to Finn, rust-for-linux, linux-fsdevel
On Sat, Jan 07, 2023 at 11:36:27AM +0100, Finn Behrens wrote:
> Hi,
> 
> I’v started to implement the proc filesystem abstractions in rust, as
> I want to use it for a driver written in rust. Currently this requires
> some rust code that is only in the rust branch, so does not apply onto
> 6.2-rc2.

Please no, no new driver should ever be using /proc at all.  Please
stick with the sysfs api which is what a driver should always be using
instead.

/proc is for processes, not devices or drivers at all.  We learned from
our mistakes 2 decades ago, please do not forget the lessons of the
past.

thanks,

greg k-h
```

### rust: Enable incremental compilation

Signed-off-by: Asahi Lina <>

[https://github.com/AsahiLinux/linux/commit/6cb6d0b4fbbe5d99e82829e9c20618f85b5d890a.patch](https://github.com/AsahiLinux/linux/commit/6cb6d0b4fbbe5d99e82829e9c20618f85b5d890a.patch)

```patch
From 6cb6d0b4fbbe5d99e82829e9c20618f85b5d890a Mon Sep 17 00:00:00 2001
From: Asahi Lina <lina@asahilina.net>
Date: Sat, 21 Jan 2023 01:26:39 +0900
Subject: [PATCH] rust: Enable incremental compilation

Signed-off-by: Asahi Lina <lina@asahilina.net>
---
 Makefile                       |  7 +++++++
 drivers/gpu/drm/asahi/Makefile |  2 +-
 scripts/Makefile.build         | 10 ++++++++++
 scripts/mod/modpost.c          |  1 +
 4 files changed, 19 insertions(+), 1 deletion(-)

diff --git a/Makefile b/Makefile
index 0992f827888dd9..17eda4de633d63 100644
--- a/Makefile
+++ b/Makefile
@@ -120,6 +120,13 @@ endif
 
 export KBUILD_CHECKSRC
 
+# Enable Rust incremental compilation.
+#
+# Use 'make RUST_INCREMENTAL=1' to enable it.
+ifeq ("$(origin RUST_INCREMENTAL)", "command line")
+  KBUILD_RUST_INCREMENTAL := $(RUST_INCREMENTAL)
+endif
+
 # Enable "clippy" (a linter) as part of the Rust compilation.
 #
 # Use 'make CLIPPY=1' to enable it.
diff --git a/drivers/gpu/drm/asahi/Makefile b/drivers/gpu/drm/asahi/Makefile
index e6724866798760..18f6169ffdccae 100644
--- a/drivers/gpu/drm/asahi/Makefile
+++ b/drivers/gpu/drm/asahi/Makefile
@@ -1,3 +1,3 @@
 # SPDX-License-Identifier: GPL-2.0
 
-obj-$(CONFIG_DRM_ASAHI) += asahi.o
+obj-$(CONFIG_DRM_ASAHI) += asahi.a
diff --git a/scripts/Makefile.build b/scripts/Makefile.build
index c46daa410d35d5..bb3b62f32ef448 100644
--- a/scripts/Makefile.build
+++ b/scripts/Makefile.build
@@ -279,6 +279,7 @@ rust_allowed_features := allocator_api,array_from_fn,bench_black_box,core_ffi_c,
 
 rust_common_cmd = \
 	RUST_MODFILE=$(modfile) $(RUSTC_OR_CLIPPY) $(rust_flags) \
+	$(if $(RUST_INCREMENTAL),-Cincremental=$(obj)/rust_incremental,) \
 	-Zallow-features=$(rust_allowed_features) \
 	-Zcrate-attr=no_std \
 	-Zcrate-attr='feature($(rust_allowed_features))' \
@@ -306,6 +307,15 @@ quiet_cmd_rustc_o_rs = $(RUSTC_OR_CLIPPY_QUIET) $(quiet_modtag) $@
 $(obj)/%.o: $(src)/%.rs FORCE
 	$(call if_changed_dep,rustc_o_rs)
 
+quiet_cmd_rustc_a_rs = $(RUSTC_OR_CLIPPY_QUIET) $(quiet_modtag) $@
+      cmd_rustc_a_rs = \
+	$(rust_common_cmd) --emit=dep-info,link -Ccodegen-units=32 $<; \
+	mv $(dir $@)/$(patsubst %.a,lib%.rlib,$(notdir $@)) $@; \
+	$(rust_handle_depfile)
+
+$(obj)/%.a: $(src)/%.rs FORCE
+	$(call if_changed_dep,rustc_a_rs)
+
 quiet_cmd_rustc_rsi_rs = $(RUSTC_OR_CLIPPY_QUIET) $(quiet_modtag) $@
       cmd_rustc_rsi_rs = \
 	$(rust_common_cmd) --emit=dep-info -Zunpretty=expanded $< >$@; \
diff --git a/scripts/mod/modpost.c b/scripts/mod/modpost.c
index 2c80da0220c326..853f0a502d1abe 100644
--- a/scripts/mod/modpost.c
+++ b/scripts/mod/modpost.c
@@ -774,6 +774,7 @@ static const char *const section_white_list[] =
 	".fmt_slot*",			/* EZchip */
 	".gnu.lto*",
 	".discard.*",
+	".rmeta",
 	NULL
 };
 

```


### [PATCH 1/5] rust: types: introduce `ScopeGuard`

```rust
Wedson Almeida Filho <>
	
Jan 19, 2023, 2:45 PM
	
This allows us to run some code when the guard is dropped (e.g.,
implicitly when it goes out of scope). We can also prevent the
guard from running by calling its `dismiss()` method.

Signed-off-by: Wedson Almeida Filho <>
---
 rust/kernel/types.rs | 127 ++++++++++++++++++++++++++++++++++++++++++-
 1 file changed, 126 insertions(+), 1 deletion(-)

diff --git a/rust/kernel/types.rs b/rust/kernel/types.rs
index e84e51ec9716..f0ad4472292d 100644
--- a/rust/kernel/types.rs
+++ b/rust/kernel/types.rs
@@ -2,7 +2,132 @@

 //! Kernel types.

-use core::{cell::UnsafeCell, mem::MaybeUninit};
+use alloc::boxed::Box;
+use core::{
+    cell::UnsafeCell,
+    mem::MaybeUninit,
+    ops::{Deref, DerefMut},
+};
+
+/// Runs a cleanup function/closure when dropped.
+///
+/// The [`ScopeGuard::dismiss`] function prevents the cleanup function from running.
+///
+/// # Examples
+///
+/// In the example below, we have multiple exit paths and we want to log regardless of which one is
+/// taken:
+/// ```
+/// # use kernel::ScopeGuard;
+/// fn example1(arg: bool) {
+///     let _log = ScopeGuard::new(|| pr_info!("example1 completed\n"));
+///
+///     if arg {
+///         return;
+///     }
+///
+///     pr_info!("Do something...\n");
+/// }
+///
+/// # example1(false);
+/// # example1(true);
+/// ```
+///
+/// In the example below, we want to log the same message on all early exits but a different one on
+/// the main exit path:
+/// ```
+/// # use kernel::ScopeGuard;
+/// fn example2(arg: bool) {
+///     let log = ScopeGuard::new(|| pr_info!("example2 returned early\n"));
+///
+///     if arg {
+///         return;
+///     }
+///
+///     // (Other early returns...)
+///
+///     log.dismiss();
+///     pr_info!("example2 no early return\n");
+/// }
+///
+/// # example2(false);
+/// # example2(true);
+/// ```
+///
+/// In the example below, we need a mutable object (the vector) to be accessible within the log
+/// function, so we wrap it in the [`ScopeGuard`]:
+/// ```
+/// # use kernel::ScopeGuard;
+/// fn example3(arg: bool) -> Result {
+///     let mut vec =
+///         ScopeGuard::new_with_data(Vec::new(), |v| pr_info!("vec had {} elements\n", v.len()));
+///
+///     vec.try_push(10u8)?;
+///     if arg {
+///         return Ok(());
+///     }
+///     vec.try_push(20u8)?;
+///     Ok(())
+/// }
+///
+/// # assert_eq!(example3(false), Ok(()));
+/// # assert_eq!(example3(true), Ok(()));
+/// ```
+///
+/// # Invariants
+///
+/// The value stored in the struct is nearly always `Some(_)`, except between
+/// [`ScopeGuard::dismiss`] and [`ScopeGuard::drop`]: in this case, it will be `None` as the value
+/// will have been returned to the caller. Since  [`ScopeGuard::dismiss`] consumes the guard,
+/// callers won't be able to use it anymore.
+pub struct ScopeGuard<T, F: FnOnce(T)>(Option<(T, F)>);
+
+impl<T, F: FnOnce(T)> ScopeGuard<T, F> {
+    /// Creates a new guarded object wrapping the given data and with the given cleanup function.
+    pub fn new_with_data(data: T, cleanup_func: F) -> Self {
+        // INVARIANT: The struct is being initialised with `Some(_)`.
+        Self(Some((data, cleanup_func)))
+    }
+
+    /// Prevents the cleanup function from running and returns the guarded data.
+    pub fn dismiss(mut self) -> T {
+        // INVARIANT: This is the exception case in the invariant; it is not visible to callers
+        // because this function consumes `self`.
+        self.0.take().unwrap().0
+    }
+}
+
+impl ScopeGuard<(), Box<dyn FnOnce(())>> {
+    /// Creates a new guarded object with the given cleanup function.
+    pub fn new(cleanup: impl FnOnce()) -> ScopeGuard<(), impl FnOnce(())> {
+        ScopeGuard::new_with_data((), move |_| cleanup())
+    }
+}
+
+impl<T, F: FnOnce(T)> Deref for ScopeGuard<T, F> {
+    type Target = T;
+
+    fn deref(&self) -> &T {
+        // The type invariants guarantee that `unwrap` will succeed.
+        &self.0.as_ref().unwrap().0
+    }
+}
+
+impl<T, F: FnOnce(T)> DerefMut for ScopeGuard<T, F> {
+    fn deref_mut(&mut self) -> &mut T {
+        // The type invariants guarantee that `unwrap` will succeed.
+        &mut self.0.as_mut().unwrap().0
+    }
+}
+
+impl<T, F: FnOnce(T)> Drop for ScopeGuard<T, F> {
+    fn drop(&mut self) {
+        // Run the cleanup function if one is still present.
+        if let Some((data, cleanup)) = self.0.take() {
+            cleanup(data)
+        }
+    }
+}
```

## ForeignOwnable

### [PATCH 2/5] rust: types: introduce `ForeignOwnable`

```rust
Wedson Almeida Filho <>
	
Thu, Jan 19, 2:46 PM
	
It was originally called `PointerWrapper`. It is used to convert
a Rust object to a pointer representation (void *) that can be
stored on the C side, used, and eventually returned to Rust.

Signed-off-by: Wedson Almeida Filho <>
---
 rust/kernel/lib.rs   |  1 +
 rust/kernel/types.rs | 54 ++++++++++++++++++++++++++++++++++++++++++++
 2 files changed, 55 insertions(+)

diff --git a/rust/kernel/lib.rs b/rust/kernel/lib.rs
index e0b0e953907d..223564f9f0cc 100644
--- a/rust/kernel/lib.rs
+++ b/rust/kernel/lib.rs
@@ -16,6 +16,7 @@
 #![feature(coerce_unsized)]
 #![feature(core_ffi_c)]
 #![feature(dispatch_from_dyn)]
+#![feature(generic_associated_types)]
 #![feature(receiver_trait)]
 #![feature(unsize)]

diff --git a/rust/kernel/types.rs b/rust/kernel/types.rs
index f0ad4472292d..5475f6163002 100644
--- a/rust/kernel/types.rs
+++ b/rust/kernel/types.rs
@@ -9,6 +9,60 @@ use core::{
     ops::{Deref, DerefMut},
 };

+/// Used to transfer ownership to and from foreign (non-Rust) languages.
+///
+/// Ownership is transferred from Rust to a foreign language by calling [`Self::into_foreign`] and
+/// later may be transferred back to Rust by calling [`Self::from_foreign`].
+///
+/// This trait is meant to be used in cases when Rust objects are stored in C objects and
+/// eventually "freed" back to Rust.
+pub trait ForeignOwnable {
+    /// Type of values borrowed between calls to [`ForeignOwnable::into_foreign`] and
+    /// [`ForeignOwnable::from_foreign`].
+    type Borrowed<'a>;
+
+    /// Converts a Rust-owned object to a foreign-owned one.
+    ///
+    /// The foreign representation is a pointer to void.
+    fn into_foreign(self) -> *const core::ffi::c_void;
+
+    /// Borrows a foreign-owned object.
+    ///
+    /// # Safety
+    ///
+    /// `ptr` must have been returned by a previous call to [`ForeignOwnable::into_foreign`] for
+    /// which a previous matching [`ForeignOwnable::from_foreign`] hasn't been called yet.
+    /// Additionally, all instances (if any) of values returned by [`ForeignOwnable::borrow_mut`]
+    /// for this object must have been dropped.
+    unsafe fn borrow<'a>(ptr: *const core::ffi::c_void) -> Self::Borrowed<'a>;
+
+    /// Mutably borrows a foreign-owned object.
+    ///
+    /// # Safety
+    ///
+    /// `ptr` must have been returned by a previous call to [`ForeignOwnable::into_foreign`] for
+    /// which a previous matching [`ForeignOwnable::from_foreign`] hasn't been called yet.
+    /// Additionally, all instances (if any) of values returned by [`ForeignOwnable::borrow`] and
+    /// [`ForeignOwnable::borrow_mut`] for this object must have been dropped.
+    unsafe fn borrow_mut<T: ForeignOwnable>(ptr: *const core::ffi::c_void) -> ScopeGuard<T, fn(T)> {
+        // SAFETY: The safety requirements ensure that `ptr` came from a previous call to
+        // `into_foreign`.
+        ScopeGuard::new_with_data(unsafe { T::from_foreign(ptr) }, |d| {
+            d.into_foreign();
+        })
+    }
+
+    /// Converts a foreign-owned object back to a Rust-owned one.
+    ///
+    /// # Safety
+    ///
+    /// `ptr` must have been returned by a previous call to [`ForeignOwnable::into_foreign`] for
+    /// which a previous matching [`ForeignOwnable::from_foreign`] hasn't been called yet.
+    /// Additionally, all instances (if any) of values returned by [`ForeignOwnable::borrow`] and
+    /// [`ForeignOwnable::borrow_mut`] for this object must have been dropped.
+    unsafe fn from_foreign(ptr: *const core::ffi::c_void) -> Self;
+}
+
 /// Runs a cleanup function/closure when dropped.
 ///
 /// The [`ScopeGuard::dismiss`] function prevents the cleanup function from running.
```

### [PATCH 3/5] rust: types: implement `ForeignOwnable` for `Box<T>`

```rust
Wedson Almeida Filho <>
	
Jan 19, 2023, 2:46 PM (18 hours ago)
	
This allows us to hand ownership of Rust dynamically allocated
objects to the C side of the kernel.

Signed-off-by: Wedson Almeida Filho <>
---
 rust/kernel/types.rs | 22 ++++++++++++++++++++++
 1 file changed, 22 insertions(+)

diff --git a/rust/kernel/types.rs b/rust/kernel/types.rs
index 5475f6163002..e037c262f23e 100644
--- a/rust/kernel/types.rs
+++ b/rust/kernel/types.rs
@@ -63,6 +63,28 @@ pub trait ForeignOwnable {
     unsafe fn from_foreign(ptr: *const core::ffi::c_void) -> Self;
 }

+impl<T: 'static> ForeignOwnable for Box<T> {
+    type Borrowed<'a> = &'a T;
+
+    fn into_foreign(self) -> *const core::ffi::c_void {
+        Box::into_raw(self) as _
+    }
+
+    unsafe fn borrow<'a>(ptr: *const core::ffi::c_void) -> &'a T {
+        // SAFETY: The safety requirements for this function ensure that the object is still alive,
+        // so it is safe to dereference the raw pointer.
+        // The safety requirements of `from_foreign` also ensure that the object remains alive for
+        // the lifetime of the returned value.
+        unsafe { &*ptr.cast() }
+    }
+
+    unsafe fn from_foreign(ptr: *const core::ffi::c_void) -> Self {
+        // SAFETY: The safety requirements of this function ensure that `ptr` comes from a previous
+        // call to `Self::into_foreign`.
+        unsafe { Box::from_raw(ptr as _) }
+    }
+}
+
 /// Runs a cleanup function/closure when dropped.
 ///
 /// The [`ScopeGuard::dismiss`] function prevents the cleanup function from running.
```


### [PATCH 4/5] rust: types: implement `ForeignOwnable` for the unit type

```rust
Wedson Almeida Filho <>
	
Thu, Jan 19, 2:45 PM
	
This allows us to use the unit type `()` when we have no object whose
ownership must be managed but one implementing the `ForeignOwnable`
trait is needed.

Signed-off-by: Wedson Almeida Filho <>
---
 rust/kernel/types.rs | 13 +++++++++++++
 1 file changed, 13 insertions(+)

diff --git a/rust/kernel/types.rs b/rust/kernel/types.rs
index e037c262f23e..8f80cffbff59 100644
--- a/rust/kernel/types.rs
+++ b/rust/kernel/types.rs
@@ -85,6 +85,19 @@ impl<T: 'static> ForeignOwnable for Box<T> {
     }
 }

+impl ForeignOwnable for () {
+    type Borrowed<'a> = ();
+
+    fn into_foreign(self) -> *const core::ffi::c_void {
+        // We use 1 to be different from a null pointer.
+        1usize as _
+    }
+
+    unsafe fn borrow<'a>(_: *const core::ffi::c_void) -> Self::Borrowed<'a> {}
+
+    unsafe fn from_foreign(_: *const core::ffi::c_void) -> Self {}
+}
+
 /// Runs a cleanup function/closure when dropped.
 ///
 /// The [`ScopeGuard::dismiss`] function prevents the cleanup function from running.
```

### [PATCH 5/5] rust: types: implement `ForeignOwnable` for `Arc<T>`

```rust
Wedson Almeida Filho <>
	
Thu, Jan 19, 3:03 PM
	
This allows us to hand ownership of Rust ref-counted objects to
the C side of the kernel.

Signed-off-by: Wedson Almeida Filho <>
---
 rust/kernel/sync/arc.rs | 32 +++++++++++++++++++++++++++++++-
 1 file changed, 31 insertions(+), 1 deletion(-)

diff --git a/rust/kernel/sync/arc.rs b/rust/kernel/sync/arc.rs
index ff73f9240ca1..519a6ec43644 100644
--- a/rust/kernel/sync/arc.rs
+++ b/rust/kernel/sync/arc.rs
@@ -15,7 +15,11 @@
 //!
 //! [`Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html

-use crate::{bindings, error::Result, types::Opaque};
+use crate::{
+    bindings,
+    error::Result,
+    types::{ForeignOwnable, Opaque},
+};
 use alloc::boxed::Box;
 use core::{
     marker::{PhantomData, Unsize},
@@ -189,6 +193,32 @@ impl<T: ?Sized> Arc<T> {
     }
 }

+impl<T: 'static> ForeignOwnable for Arc<T> {
+    type Borrowed<'a> = ArcBorrow<'a, T>;
+
+    fn into_foreign(self) -> *const core::ffi::c_void {
+        ManuallyDrop::new(self).ptr.as_ptr() as _
+    }
+
+    unsafe fn borrow<'a>(ptr: *const core::ffi::c_void) -> ArcBorrow<'a, T> {
+        // SAFETY: By the safety requirement of this function, we know that `ptr` came from
+        // a previous call to `Arc::into_foreign`.
+        let inner = NonNull::new(ptr as *mut ArcInner<T>).unwrap();
+
+        // SAFETY: The safety requirements of `from_foreign` ensure that the object remains alive
+        // for the lifetime of the returned value. Additionally, the safety requirements of
+        // `ForeignOwnable::borrow_mut` ensure that no new mutable references are created.
+        unsafe { ArcBorrow::new(inner) }
+    }
+
+    unsafe fn from_foreign(ptr: *const core::ffi::c_void) -> Self {
+        // SAFETY: By the safety requirement of this function, we know that `ptr` came from
+        // a previous call to `Arc::into_foreign`, which owned guarantees that `ptr` is valid and
+        // owns a reference.
+        unsafe { Self::from_inner(NonNull::new(ptr as _).unwrap()) }
+    }
+}
+
 impl<T: ?Sized> Deref for Arc<T> {
     type Target = T;

```


## GPU Driver written in Rust

[https://github.com/AsahiLinux/linux/tree/asahi/drivers/gpu/drm/asahi](https://github.com/AsahiLinux/linux/tree/asahi/drivers/gpu/drm/asahi)

[https://github.com/AsahiLinux/linux/blob/asahi/drivers/gpu/drm/asahi/gpu.rs](https://github.com/AsahiLinux/linux/blob/asahi/drivers/gpu/drm/asahi/gpu.rs)




