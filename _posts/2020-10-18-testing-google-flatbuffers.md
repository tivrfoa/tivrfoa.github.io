---
layout: post
title:  "Testing Google FlatBuffers"
date:   2020-10-18 18:00:00 -0300
categories: api flatbuffers google java rust
---

### Code repository

[https://github.com/tivrfoa/flatbuffers-java-rust](https://github.com/tivrfoa/flatbuffers-java-rust)

### Building the Jar file


Clone the project:

```sh
git clone https://github.com/google/flatbuffers.git
```

I needed to disable maven-gpg-plugin:

```diff
diff --git a/pom.xml b/pom.xml
index b9153383..591a25f4 100644
--- a/pom.xml
+++ b/pom.xml
@@ -119,6 +119,7 @@
               <goal>sign</goal>
             </goals>
             <configuration>
+                               <skip>true</skip>
                 <gpgArguments>
```

Then run Maven install with Java 8:

```sh
mvn install
```

### Running with JBang

In order to run SampleBinary.java with JBang, you need to the following in the beginning of the file:

```java
//DEPS com.google.flatbuffers:flatbuffers-java:1.12.0
//
//SOURCES MyGame/Sample/Color.java
//SOURCES MyGame/Sample/Equipment.java
//SOURCES MyGame/Sample/Monster.java
//SOURCES MyGame/Sample/Vec3.java
//SOURCES MyGame/Sample/Weapon.java

import MyGame.Sample.Color;
import MyGame.Sample.Equipment;
import MyGame.Sample.Monster;
import MyGame.Sample.Vec3;
import MyGame.Sample.Weapon;

import com.google.flatbuffers.FlatBufferBuilder;

import java.nio.ByteBuffer;

import java.io.FileOutputStream;
import java.nio.channels.FileChannel;

class SampleBinary {
  // Example how to use FlatBuffers to create and read binary buffers.
  public static void main(String[] args) {
    FlatBufferBuilder builder = new FlatBufferBuilder(0);

    // Create some weapons for our Monster ('Sword' and 'Axe').
    int weaponOneName = builder.createString("Sword");
    short weaponOneDamage = 3;
    int weaponTwoName = builder.createString("Axe");
    short weaponTwoDamage = 5;

    // Use the `createWeapon()` helper function to create the weapons, since we set every field.
    int[] weaps = new int[2];
    weaps[0] = Weapon.createWeapon(builder, weaponOneName, weaponOneDamage);
    weaps[1] = Weapon.createWeapon(builder, weaponTwoName, weaponTwoDamage);

    // Serialize the FlatBuffer data.
    int name = builder.createString("Orc");
    byte[] treasure = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    int inv = Monster.createInventoryVector(builder, treasure);
    int weapons = Monster.createWeaponsVector(builder, weaps);
    int pos = Vec3.createVec3(builder, 1.0f, 2.0f, 3.0f);

    Monster.startMonster(builder);
    Monster.addPos(builder, pos);
    Monster.addName(builder, name);
    Monster.addColor(builder, Color.Red);
    Monster.addHp(builder, (short)300);
    Monster.addInventory(builder, inv);
    Monster.addWeapons(builder, weapons);
    Monster.addEquippedType(builder, Equipment.Weapon);
    Monster.addEquipped(builder, weaps[1]);
    int orc = Monster.endMonster(builder);

    builder.finish(orc); // You could also call `Monster.finishMonsterBuffer(builder, orc);`.

    // We now have a FlatBuffer that can be stored on disk or sent over a network.

    // ...Code to store to disk or send over a network goes here...
    ByteBuffer buf = builder.dataBuffer();
    ByteBuffer bufCopy = buf.duplicate();

	try {
		FileChannel fc = new FileOutputStream("../flatbuffer.data").getChannel();
		fc.write(buf);
		fc.close();
	} catch (Exception e) { e.printStackTrace(); }

	buf = bufCopy;

    // Get access to the root:
    Monster monster = Monster.getRootAsMonster(buf);

    // Note: We did not set the `mana` field explicitly, so we get back the default value.
    assert monster.mana() == (short)150;
    assert monster.hp() == (short)300;
    assert monster.name().equals("Orc");
    assert monster.color() == Color.Red;
    assert monster.pos().x() == 1.0f;
    assert monster.pos().y() == 2.0f;
    assert monster.pos().z() == 3.0f;

    // Get and test the `inventory` FlatBuffer `vector`.
    for (int i = 0; i < monster.inventoryLength(); i++) {
      assert monster.inventory(i) == (byte)i;
    }

    // Get and test the `weapons` FlatBuffer `vector` of `table`s.
    String[] expectedWeaponNames = {"Sword", "Axe"};
    int[] expectedWeaponDamages = {3, 5};
    for (int i = 0; i < monster.weaponsLength(); i++) {
      assert monster.weapons(i).name().equals(expectedWeaponNames[i]);
      assert monster.weapons(i).damage() == expectedWeaponDamages[i];
    }

    Weapon.Vector weaponsVector = monster.weaponsVector();
    for (int i = 0; i < weaponsVector.length(); i++) {
      assert weaponsVector.get(i).name().equals(expectedWeaponNames[i]);
      assert weaponsVector.get(i).damage() == expectedWeaponDamages[i];
    }

    // Get and test the `equipped` FlatBuffer `union`.
    assert monster.equippedType() == Equipment.Weapon;
    Weapon equipped = (Weapon)monster.equipped(new Weapon());
    assert equipped.name().equals("Axe");
    assert equipped.damage() == 5;

    System.out.println("The FlatBuffer was successfully created and verified!");
  }
}

```

Then run to create the file:

```sh
java$ jbang SampleBinary.java 
The FlatBuffer was successfully created and verified!
```

### Now let's read the file with Rust

```sh
rust$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/testflatbuffers`
300
150
Some("Huge Orc")
```

### Testing from github rust repo

[https://github.com/google/flatbuffers/tree/master/tests/rust_usage_test](https://github.com/google/flatbuffers/tree/master/tests/rust_usage_test)

```sh
$ cargo run --bin monster_example
```

```log
80
150
"MyMonster"
```

### References

[FlatBuffers - Use in Java](http://google.github.io/flatbuffers/flatbuffers_guide_use_java.html)
[failed to execute goal org.apache.maven.plugins:maven-gpg-plugin](https://stackoverflow.com/a/32112910/339561)
[FlatBuffers Java API](https://mvnrepository.com/artifact/com.google.flatbuffers/flatbuffers-java)
[ByteBuffer.duplicate](https://docs.oracle.com/javase/9/docs/api/java/nio/ByteBuffer.html#duplicate--)
[monster_example.rs](https://github.com/google/flatbuffers/blob/master/tests/rust_usage_test/bin/monster_example.rs)
[flatbuffers crate](https://crates.io/crates/flatbuffers)

Chai T. Rex
>You have two mod my_games. One in main.rs. One in my_game.rs. You should have only one in the parent module of the module you're declaring (here, main.rs is the parent module).
