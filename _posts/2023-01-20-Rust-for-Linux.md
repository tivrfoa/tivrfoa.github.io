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

## Use sysfs api instead of /proc


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

## rust: Enable incremental compilation

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


## [PATCH 1/5] rust: types: introduce `ScopeGuard`

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

## [PATCH 4/7] rust: sync: introduce `ArcBorrow`

```rust
Wedson Almeida Filho <>
	
Dec 28, 2022, 3:05 AM
	
This allows us to create references to a ref-counted allocation without
double-indirection and that still allow us to increment the refcount to
a new `Arc<T>`.

Signed-off-by: Wedson Almeida Filho <wedsonaf@gmail.com>
---
 rust/kernel/sync.rs     |  2 +-
 rust/kernel/sync/arc.rs | 97 +++++++++++++++++++++++++++++++++++++++++
 2 files changed, 98 insertions(+), 1 deletion(-)

diff --git a/rust/kernel/sync.rs b/rust/kernel/sync.rs
index 39b379dd548f..5de03ea83ea1 100644
--- a/rust/kernel/sync.rs
+++ b/rust/kernel/sync.rs
@@ -7,4 +7,4 @@

 mod arc;

-pub use arc::Arc;
+pub use arc::{Arc, ArcBorrow};
diff --git a/rust/kernel/sync/arc.rs b/rust/kernel/sync/arc.rs
index dbc7596cc3ce..f68bfc02c81a 100644
--- a/rust/kernel/sync/arc.rs
+++ b/rust/kernel/sync/arc.rs
@@ -19,6 +19,7 @@ use crate::{bindings, error::Result, types::Opaque};
 use alloc::boxed::Box;
 use core::{
     marker::{PhantomData, Unsize},
+    mem::ManuallyDrop,
     ops::Deref,
     ptr::NonNull,
 };
@@ -164,6 +165,18 @@ impl<T: ?Sized> Arc<T> {
             _p: PhantomData,
         }
     }
+
+    /// Returns an [`ArcBorrow`] from the given [`Arc`].
+    ///
+    /// This is useful when the argument of a function call is an [`ArcBorrow`] (e.g., in a method
+    /// receiver), but we have an [`Arc`] instead. Getting an [`ArcBorrow`] is free when optimised.
+    #[inline]
+    pub fn as_arc_borrow(&self) -> ArcBorrow<'_, T> {
+        // SAFETY: The constraint that the lifetime of the shared reference must outlive that of
+        // the returned `ArcBorrow` ensures that the object remains alive and that no mutable
+        // reference can be created.
+        unsafe { ArcBorrow::new(self.ptr) }
+    }
 }

 impl<T: ?Sized> Deref for Arc<T> {
@@ -208,3 +221,87 @@ impl<T: ?Sized> Drop for Arc<T> {
         }
     }
 }
+
+/// A borrowed reference to an [`Arc`] instance.
+///
+/// For cases when one doesn't ever need to increment the refcount on the allocation, it is simpler
+/// to use just `&T`, which we can trivially get from an `Arc<T>` instance.
+///
+/// However, when one may need to increment the refcount, it is preferable to use an `ArcBorrow<T>`
+/// over `&Arc<T>` because the latter results in a double-indirection: a pointer (shared reference)
+/// to a pointer (`Arc<T>`) to the object (`T`). An [`ArcBorrow`] eliminates this double
+/// indirection while still allowing one to increment the refcount and getting an `Arc<T>` when/if
+/// needed.
+///
+/// # Invariants
+///
+/// There are no mutable references to the underlying [`Arc`], and it remains valid for the
+/// lifetime of the [`ArcBorrow`] instance.
+///
+/// # Example
+///
+/// ```
+/// use crate::sync::{Arc, ArcBorrow};
+///
+/// struct Example;
+///
+/// fn do_something(e: ArcBorrow<'_, Example>) -> Arc<Example> {
+///     e.into()
+/// }
+///
+/// let obj = Arc::try_new(Example)?;
+/// let cloned = do_something(obj.as_arc_borrow());
+///
+/// // Assert that both `obj` and `cloned` point to the same underlying object.
+/// assert!(core::ptr::eq(&*obj, &*cloned));
+/// ```
+pub struct ArcBorrow<'a, T: ?Sized + 'a> {
+    inner: NonNull<ArcInner<T>>,
+    _p: PhantomData<&'a ()>,
+}
+
+impl<T: ?Sized> Clone for ArcBorrow<'_, T> {
+    fn clone(&self) -> Self {
+        *self
+    }
+}
+
+impl<T: ?Sized> Copy for ArcBorrow<'_, T> {}
+
+impl<T: ?Sized> ArcBorrow<'_, T> {
+    /// Creates a new [`ArcBorrow`] instance.
+    ///
+    /// # Safety
+    ///
+    /// Callers must ensure the following for the lifetime of the returned [`ArcBorrow`] instance:
+    /// 1. That `inner` remains valid;
+    /// 2. That no mutable references to `inner` are created.
+    unsafe fn new(inner: NonNull<ArcInner<T>>) -> Self {
+        // INVARIANT: The safety requirements guarantee the invariants.
+        Self {
+            inner,
+            _p: PhantomData,
+        }
+    }
+}
+
+impl<T: ?Sized> From<ArcBorrow<'_, T>> for Arc<T> {
+    fn from(b: ArcBorrow<'_, T>) -> Self {
+        // SAFETY: The existence of `b` guarantees that the refcount is non-zero. `ManuallyDrop`
+        // guarantees that `drop` isn't called, so it's ok that the temporary `Arc` doesn't own the
+        // increment.
+        ManuallyDrop::new(unsafe { Arc::from_inner(b.inner) })
+            .deref()
+            .clone()
+    }
+}
+
+impl<T: ?Sized> Deref for ArcBorrow<'_, T> {
+    type Target = T;
+
+    fn deref(&self) -> &Self::Target {
+        // SAFETY: By the type invariant, the underlying object is still alive with no mutable
+        // references to it, so it is safe to create a shared reference.
+        unsafe { &self.inner.as_ref().data }
+    }
+}
```


```txt
Boqun Feng <>
	
Jan 20, 2023, 9:55 PM
	
Hi,
Somehow I run into this code again, and after a few thoughts, I'm
wondering: isn't ArcBorrow just &ArcInner<T>?

I've tried the following diff, and it seems working. The better parts
are 1) #[derive(Clone, Copy)] works and 2) I'm able to remove a few code
including one "unsafe" in ArcBorrow::deref().

But sure, more close look is needed. Thoughts?
```

## [PATCH 1/7] rust: sync: add `Arc` for ref-counted allocations

```rust
Wedson Almeida Filho <>
	
Dec 28, 2022, 3:17 AM
	
This is a basic implementation of `Arc` backed by C's `refcount_t`. It
allows Rust code to idiomatically allocate memory that is ref-counted.

Cc: Will Deacon <will@kernel.org>
Cc: Peter Zijlstra <peterz@infradead.org>
Cc: Boqun Feng <boqun.feng@gmail.com>
Cc: Mark Rutland <mark.rutland@arm.com>
Signed-off-by: Wedson Almeida Filho <wedsonaf@gmail.com>
---
 rust/bindings/bindings_helper.h |   1 +
 rust/bindings/lib.rs            |   1 +
 rust/helpers.c                  |  19 ++++
 rust/kernel/lib.rs              |   1 +
 rust/kernel/sync.rs             |  10 ++
 rust/kernel/sync/arc.rs         | 157 ++++++++++++++++++++++++++++++++
 6 files changed, 189 insertions(+)
 create mode 100644 rust/kernel/sync.rs
 create mode 100644 rust/kernel/sync/arc.rs

diff --git a/rust/bindings/bindings_helper.h b/rust/bindings/bindings_helper.h
index c48bc284214a..75d85bd6c592 100644
--- a/rust/bindings/bindings_helper.h
+++ b/rust/bindings/bindings_helper.h
@@ -7,6 +7,7 @@
  */

 #include <linux/slab.h>
+#include <linux/refcount.h>

 /* `bindgen` gets confused at certain things. */
 const gfp_t BINDINGS_GFP_KERNEL = GFP_KERNEL;
diff --git a/rust/bindings/lib.rs b/rust/bindings/lib.rs
index 6c50ee62c56b..7b246454e009 100644
--- a/rust/bindings/lib.rs
+++ b/rust/bindings/lib.rs
@@ -41,6 +41,7 @@ mod bindings_raw {
 #[allow(dead_code)]
 mod bindings_helper {
     // Import the generated bindings for types.
+    use super::bindings_raw::*;
     include!(concat!(
         env!("OBJTREE"),
         "/rust/bindings/bindings_helpers_generated.rs"
diff --git a/rust/helpers.c b/rust/helpers.c
index b4f15eee2ffd..09a4d93f9d62 100644
--- a/rust/helpers.c
+++ b/rust/helpers.c
@@ -20,6 +20,7 @@

 #include <linux/bug.h>
 #include <linux/build_bug.h>
+#include <linux/refcount.h>

 __noreturn void rust_helper_BUG(void)
 {
@@ -27,6 +28,24 @@ __noreturn void rust_helper_BUG(void)
 }
 EXPORT_SYMBOL_GPL(rust_helper_BUG);

+refcount_t rust_helper_REFCOUNT_INIT(int n)
+{
+       return (refcount_t)REFCOUNT_INIT(n);
+}
+EXPORT_SYMBOL_GPL(rust_helper_REFCOUNT_INIT);
+
+void rust_helper_refcount_inc(refcount_t *r)
+{
+       refcount_inc(r);
+}
+EXPORT_SYMBOL_GPL(rust_helper_refcount_inc);
+
+bool rust_helper_refcount_dec_and_test(refcount_t *r)
+{
+       return refcount_dec_and_test(r);
+}
+EXPORT_SYMBOL_GPL(rust_helper_refcount_dec_and_test);
+
 /*
  * We use `bindgen`'s `--size_t-is-usize` option to bind the C `size_t` type
  * as the Rust `usize` type, so we can use it in contexts where Rust
diff --git a/rust/kernel/lib.rs b/rust/kernel/lib.rs
index 53040fa9e897..ace064a3702a 100644
--- a/rust/kernel/lib.rs
+++ b/rust/kernel/lib.rs
@@ -31,6 +31,7 @@ mod static_assert;
 #[doc(hidden)]
 pub mod std_vendor;
 pub mod str;
+pub mod sync;
 pub mod types;

 #[doc(hidden)]
diff --git a/rust/kernel/sync.rs b/rust/kernel/sync.rs
new file mode 100644
index 000000000000..39b379dd548f
--- /dev/null
+++ b/rust/kernel/sync.rs
@@ -0,0 +1,10 @@
+// SPDX-License-Identifier: GPL-2.0
+
+//! Synchronisation primitives.
+//!
+//! This module contains the kernel APIs related to synchronisation that have been ported or
+//! wrapped for usage by Rust code in the kernel.
+
+mod arc;
+
+pub use arc::Arc;
diff --git a/rust/kernel/sync/arc.rs b/rust/kernel/sync/arc.rs
new file mode 100644
index 000000000000..22290eb5ab9b
--- /dev/null
+++ b/rust/kernel/sync/arc.rs
@@ -0,0 +1,157 @@
+// SPDX-License-Identifier: GPL-2.0
+
+//! A reference-counted pointer.
+//!
+//! This module implements a way for users to create reference-counted objects and pointers to
+//! them. Such a pointer automatically increments and decrements the count, and drops the
+//! underlying object when it reaches zero. It is also safe to use concurrently from multiple
+//! threads.
+//!
+//! It is different from the standard library's [`Arc`] in a few ways:
+//! 1. It is backed by the kernel's `refcount_t` type.
+//! 2. It does not support weak references, which allows it to be half the size.
+//! 3. It saturates the reference count instead of aborting when it goes over a threshold.
+//! 4. It does not provide a `get_mut` method, so the ref counted object is pinned.
+//!
+//! [`Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html
+
+use crate::{bindings, error::Result, types::Opaque};
+use alloc::boxed::Box;
+use core::{marker::PhantomData, ops::Deref, ptr::NonNull};
+
+/// A reference-counted pointer to an instance of `T`.
+///
+/// The reference count is incremented when new instances of [`Arc`] are created, and decremented
+/// when they are dropped. When the count reaches zero, the underlying `T` is also dropped.
+///
+/// # Invariants
+///
+/// The reference count on an instance of [`Arc`] is always non-zero.
+/// The object pointed to by [`Arc`] is always pinned.
+///
+/// # Examples
+///
+/// ```
+/// use kernel::sync::Arc;
+///
+/// struct Example {
+///     a: u32,
+///     b: u32,
+/// }
+///
+/// // Create a ref-counted instance of `Example`.
+/// let obj = Arc::try_new(Example { a: 10, b: 20 })?;
+///
+/// // Get a new pointer to `obj` and increment the refcount.
+/// let cloned = obj.clone();
+///
+/// // Assert that both `obj` and `cloned` point to the same underlying object.
+/// assert!(core::ptr::eq(&*obj, &*cloned));
+///
+/// // Destroy `obj` and decrement its refcount.
+/// drop(obj);
+///
+/// // Check that the values are still accessible through `cloned`.
+/// assert_eq!(cloned.a, 10);
+/// assert_eq!(cloned.b, 20);
+///
+/// // The refcount drops to zero when `cloned` goes out of scope, and the memory is freed.
+/// ```
+pub struct Arc<T: ?Sized> {
+    ptr: NonNull<ArcInner<T>>,
+    _p: PhantomData<ArcInner<T>>,
+}
+
+#[repr(C)]
+struct ArcInner<T: ?Sized> {
+    refcount: Opaque<bindings::refcount_t>,
+    data: T,
+}
+
+// SAFETY: It is safe to send `Arc<T>` to another thread when the underlying `T` is `Sync` because
+// it effectively means sharing `&T` (which is safe because `T` is `Sync`); additionally, it needs
+// `T` to be `Send` because any thread that has an `Arc<T>` may ultimately access `T` directly, for
+// example, when the reference count reaches zero and `T` is dropped.
+unsafe impl<T: ?Sized + Sync + Send> Send for Arc<T> {}
+
+// SAFETY: It is safe to send `&Arc<T>` to another thread when the underlying `T` is `Sync` for the
+// same reason as above. `T` needs to be `Send` as well because a thread can clone an `&Arc<T>`
+// into an `Arc<T>`, which may lead to `T` being accessed by the same reasoning as above.
+unsafe impl<T: ?Sized + Sync + Send> Sync for Arc<T> {}
+
+impl<T> Arc<T> {
+    /// Constructs a new reference counted instance of `T`.
+    pub fn try_new(contents: T) -> Result<Self> {
+        // INVARIANT: The refcount is initialised to a non-zero value.
+        let value = ArcInner {
+            // SAFETY: There are no safety requirements for this FFI call.
+            refcount: Opaque::new(unsafe { bindings::REFCOUNT_INIT(1) }),
+            data: contents,
+        };
+
+        let inner = Box::try_new(value)?;
+
+        // SAFETY: We just created `inner` with a reference count of 1, which is owned by the new
+        // `Arc` object.
+        Ok(unsafe { Self::from_inner(Box::leak(inner).into()) })
+    }
+}
+
+impl<T: ?Sized> Arc<T> {
+    /// Constructs a new [`Arc`] from an existing [`ArcInner`].
+    ///
+    /// # Safety
+    ///
+    /// The caller must ensure that `inner` points to a valid location and has a non-zero reference
+    /// count, one of which will be owned by the new [`Arc`] instance.
+    unsafe fn from_inner(inner: NonNull<ArcInner<T>>) -> Self {
+        // INVARIANT: By the safety requirements, the invariants hold.
+        Arc {
+            ptr: inner,
+            _p: PhantomData,
+        }
+    }
+}
+
+impl<T: ?Sized> Deref for Arc<T> {
+    type Target = T;
+
+    fn deref(&self) -> &Self::Target {
+        // SAFETY: By the type invariant, there is necessarily a reference to the object, so it is
+        // safe to dereference it.
+        unsafe { &self.ptr.as_ref().data }
+    }
+}
+
+impl<T: ?Sized> Clone for Arc<T> {
+    fn clone(&self) -> Self {
+        // INVARIANT: C `refcount_inc` saturates the refcount, so it cannot overflow to zero.
+        // SAFETY: By the type invariant, there is necessarily a reference to the object, so it is
+        // safe to increment the refcount.
+        unsafe { bindings::refcount_inc(self.ptr.as_ref().refcount.get()) };
+
+        // SAFETY: We just incremented the refcount. This increment is now owned by the new `Arc`.
+        unsafe { Self::from_inner(self.ptr) }
+    }
+}
+
+impl<T: ?Sized> Drop for Arc<T> {
+    fn drop(&mut self) {
+        // SAFETY: By the type invariant, there is necessarily a reference to the object. We cannot
+        // touch `refcount` after it's decremented to a non-zero value because another thread/CPU
+        // may concurrently decrement it to zero and free it. It is ok to have a raw pointer to
+        // freed/invalid memory as long as it is never dereferenced.
+        let refcount = unsafe { self.ptr.as_ref() }.refcount.get();
+
+        // INVARIANT: If the refcount reaches zero, there are no other instances of `Arc`, and
+        // this instance is being dropped, so the broken invariant is not observable.
+        // SAFETY: Also by the type invariant, we are allowed to decrement the refcount.
+        let is_zero = unsafe { bindings::refcount_dec_and_test(refcount) };
+        if is_zero {
+            // The count reached zero, we must free the memory.
+            //
+            // SAFETY: The pointer was initialised from the result of `Box::leak`.
+            unsafe { Box::from_raw(self.ptr.as_ptr()) };
+        }
+    }
+}

```


## GPU Driver written in Rust

[https://github.com/AsahiLinux/linux/tree/asahi/drivers/gpu/drm/asahi](https://github.com/AsahiLinux/linux/tree/asahi/drivers/gpu/drm/asahi)

[https://github.com/AsahiLinux/linux/blob/asahi/drivers/gpu/drm/asahi/gpu.rs](https://github.com/AsahiLinux/linux/blob/asahi/drivers/gpu/drm/asahi/gpu.rs)




