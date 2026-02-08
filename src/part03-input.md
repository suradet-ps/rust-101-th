## Part 03: Input และ Error Handling

ใน Part 00 ผมได้สัญญาไว้ว่าในที่สุดเราจะแทนที่ `read_vec` ด้วยฟังก์ชันที่ถามผู้ใช้ให้ป้อนตัวเลขจริงๆ น่าเสียดายที่ I/O เป็นหัวข้อที่ค่อนข้างซับซ้อน โค้ดที่ทำแบบนั้นจึงไม่ค่อยสวยสักเท่าไหร่ แต่ก็ไม่เป็นไร มาผ่านมันไปให้ได้กันเถอะ

## การอ่าน Input จากผู้ใช้

I/O มีให้ใช้งานผ่านโมดูล `std::io` ดังนั้นเราต้องนำเข้ามันด้วยคำสั่ง `use` ก่อน นอกจากนี้เรายังนำเข้า I/O prelude (ชุดคำสั่งพื้นฐาน) ซึ่งจะทำให้สิ่งต่างๆ ที่ใช้บ่อยใน I/O พร้อมใช้งานได้ทันที

```rust
use std::io::prelude::*;
use std::io;
```

มาดูฟังก์ชันนี้ทีละบรรทัดกัน อันดับแรก เราเรียกคอนสตรักเตอร์ของ `Vec` เพื่อสร้างเวกเตอร์ว่าง ตามที่กล่าวไว้ในบทก่อนหน้า `new` ที่นี่เป็นแค่ฟังก์ชันสแตติกธรรมดาที่ไม่มีอะไรพิเศษ แม้จะสามารถเรียก `new` โดยระบุไทป์ได้โดยตรง (`Vec::<i32>::new()`) แต่วิธีที่นิยมกันคือกำกับไทป์ไว้ที่ตัวแปรแทน เพราะตัวแปรนี้ต่างหากที่เราจะใช้งานตลอดทั้งฟังก์ชัน การมีไทป์ของมันอยู่ตรงนั้น (และมองเห็นได้ชัด) จึงมีประโยชน์กว่ามาก ถ้าเราไม่รู้ว่า `Vec::new` คืนค่าอะไร การไประบุพารามิเตอร์ไทป์ของมันก็ไม่ได้ช่วยอะไรเราเท่าไหร่

```rust
fn read_vec() -> Vec<i32> {
    let mut vec: Vec<i32> = Vec::new();
```

ตัวจัดการหลักสำหรับอินพุตมาตรฐานเรียกใช้ได้จากฟังก์ชัน `io::stdin()` ซึ่งคืนค่าเป็น `Stdin` - handle สำหรับอ่านข้อมูลจากผู้ใช้

```rust
    let stdin = io::stdin();
    println!("Enter a list of numbers, one per line. End with Ctrl-D (Linux) or Ctrl-Z (Windows).");
```

ตอนนี้เราต้องการวนอ่านอินพุตมาตรฐานทีละบรรทัด ใช้ลูป `for` ได้ แต่มีเรื่องที่ต้องระวัง `stdin()` คืนค่า handle ที่อาจถูกใช้ร่วมกันหลายที่ในโปรแกรม ดังนั้นเราต้อง `lock()` เพื่อให้การอ่านเป็น atomic (ไม่มีใครมาขั้นระหว่างอ่าน) ซึ่งช่วยให้ thread-safe และประสิทธิภาพดีขึ้นเมื่ออ่านหลายบรรทัด อ่าน[เอกสาร](https://doc.rust-lang.org/stable/std/io/struct.Stdin.html) เพิ่มเติม

```rust
    for line in stdin.lock().lines() {
```

## Result<T, E> - การจัดการข้อผิดพลาด

ไทป์ของตัวแปร `line` ของเรายังไม่ใช่ `String` มันเป็นไทป์ `io::Result<String>`

> **🔍 `Result<T, E>` vs `Option<T>`:**
>
> - `Option<T>` = มีค่า (`Some`) หรือ ไม่มีค่า (`None`) - ใช้เมื่อความล้มเหลวเป็นเรื่องปกติ
> - `Result<T, E>` = สำเร็จ (`Ok(T)`) หรือ ล้มเหลว (`Err(E)`) - ใช้เมื่อต้องการรู้ว่าทำไมถึงล้มเหลว
>
> ใช้ `Result` เมื่อต้องการรู้สาเหตุของข้อผิดพลาด (เช่น อ่านไฟล์ไม่ได้เพราะไม่มีไฟล์ หรือ parse ไม่ได้เพราะไม่ใช่ตัวเลข)

`io::Result` จริงๆ แล้วเป็นแค่อีกชื่อหนึ่งของ `Result` ที่กำหนดชนิดข้อผิดพลาดเฉพาะสำหรับ I/O ให้คลิกไปดู[เอกสาร](https://doc.rust-lang.org/stable/std/result/index.html)ตรงนั้นก็จะเจอรายชื่อคอนสตรักเตอร์และเมธอดทั้งหมดของไทป์นี้

### การจัดการ Result: unwrap และ match

เราจะทำแบบง่ายๆ ก่อน สมมติว่าไม่มีอะไรผิดพลาด เมธอด `unwrap` จะคืนค่า `String` ให้ถ้ามี แต่ถ้าไม่มีโปรแกรมก็จะแพนิก (panic) เนื่องจาก `Result` มีรายละเอียดของข้อผิดพลาดอยู่ด้วย ก็จะได้ข้อความแสดงข้อผิดพลาดที่พอทนดูได้

> **⚠️ ระวัง `unwrap`:** ในโค้ดจริง หลีกเลี่ยง `unwrap` เพราะจะทำให้โปรแกรมแพนิก (crash) ถ้ามีข้อผิดพลาด
> ใช้ `match` หรือ method อื่นๆ เช่น `expect("ข้อความ")` ที่ให้ข้อความ error ที่เข้าใจง่ายกว่า

ผมเลือกใช้ชื่อเดิม (`line`) สำหรับตัวแปรใหม่ เพื่อให้แน่ใจว่าจะไม่ไปใช้ `line` เก่าโดยไม่ตั้งใจ (shadowing)

```rust
        let line = line.unwrap();
```

## การแปลงข้อมูล

ตอนนี้เรามี `String` แล้ว เราอยากแปลงมันเป็น `i32` เริ่มด้วยการ `trim` บรรทัดเพื่อตัดช่องว่างหน้าหลังออก เมธอด `parse` บน `String` สามารถแปลงสตริงเป็นอะไรก็ได้ ลองหาดูสิว่าเอกสารของมันอยู่ตรงไหน

ในกรณีนี้ Rust หาเองได้ว่าเราต้องการ `i32` (จากชนิดข้อมูลที่ฟังก์ชันคืนค่า) แต่นั่นมันดูเหมือนเวทมนตร์ไปหน่อยสำหรับผม เราจะระบุให้ชัดเจนกว่านี้ `parse::<i32>`

> **💡 Turbofish `::<>`:** เมื่อ Rust ไม่สามารถอนุมาน type ได้ หรือเราอยากระบุชัดเจน
> ใช้ `::<Type>` หลังชื่อฟังก์ชัน เช่น `parse::<i32>()` หรือ `Vec::<i32>::new()`
> รูปร่างคล้ายปลา (fish) จึงเรียกว่า "turbofish syntax"

```rust
        match line.trim().parse::<i32>() {
```

`parse` คืนค่าเป็น `Result` เหมือนกัน และครั้งนี้เราใช้ `match` จัดการข้อผิดพลาด (เช่น ผู้ใช้พิมพ์อะไรที่ไม่ใช่ตัวเลข) นี่เป็นรูปแบบที่เจอบ่อยใน Rust การทำอะไรที่อาจผิดพลาดได้จะคืนค่าเป็น `Option` หรือ `Result` วิธีเดียวที่จะได้ค่าที่เราต้องการคือผ่านการจับคู่รูปแบบ (หรือฟังก์ชันช่วยอย่าง `unwrap`) ถ้าเราเรียกฟังก์ชันที่คืนค่า `Result` แล้วทิ้งค่าที่ได้ไป คอมไพเลอร์จะเตือน เราจึงไม่มีทางลืมจัดการข้อผิดพลาด หรือไปใช้ค่าที่ไม่มีความหมายโดยไม่รู้ตัวเพราะมันเกิดจากข้อผิดพลาด

```rust
            Ok(num) => {
                vec.push(num)
            },
```

เราไม่สนใจว่าเป็นข้อผิดพลาดแบบไหน ก็เลยใช้ `_` ไม่สนใจมัน

```rust
            Err(_) => {
                println!("What did I say about numbers?")
            },
        }
    }
    vec
}
```

จบแล้วสำหรับ `read_vec` ถ้ายังมีคำถามอะไร ลองไปดูเอกสารของฟังก์ชันที่เกี่ยวข้องน่าจะช่วยได้เยอะ ลองหาเอกสารของ `Vec::push` ดูสิ ผมจะไม่ให้ลิงก์ทุกครั้ง เพราะเอกสารค่อนข้างหาง่ายและคุณควรจะคุ้นเคยกับมัน

## การจัดการ Error แบบสมบูรณ์ (ทางเลือก)

สำหรับผู้ที่สนใจ นี่คือวิธีจัดการ error แบบที่โปรแกรมไม่แพนิก และแจ้งข้อผิดพลาดได้สมบูรณ์

```rust
fn read_vec_safe() -> Result<Vec<i32>, Box<dyn std::error::Error>> {
    let mut vec = Vec::new();
    let stdin = io::stdin();
    
    for line in stdin.lock().lines() {
        let line = line?; // ? operator: ถ้า Err ให้ return ทันที
        match line.trim().parse::<i32>() {
            Ok(num) => vec.push(num),
            Err(e) => println!("Parse error '{}': {}", line, e),
        }
    }
    
    Ok(vec)
}

// ใช้งาน
match read_vec_safe() {
    Ok(vec) => {
        let min = vec_min(vec);
        min.print();
    }
    Err(e) => println!("Error: {}", e),
}
```

> **💡 `?` operator:** ใช้กับ `Result` เพื่อ "unwrap ถ้า Ok, return Err ทันทีถ้าไม่ใช่"
> ทำให้โค้ดสะอาดขึ้นเมื่อมีการเรียกฟังก์ชันที่คืน `Result` หลายต่อ

## การเชื่อมโยงกับบทก่อน

ส่วนโค้ดที่เหลือ เรานำ Part 02 มาใช้ซ้ำด้วยการ import ผ่านคำสั่ง `use` ผมได้แอบใส่ `pub` ไว้หลายจุดใน Part 02 เพื่อให้ทำแบบนี้ได้ เพราะมีแค่สิ่งที่ประกาศเป็น public เท่านั้นที่ import ไปใช้ที่อื่นได้

> **💡 โน้ต:** ในบริบทของ mdbook นี้ โค้ดจากบทก่อนอยู่ใน module `part02`
> หากคุณเขียนในไฟล์เดียวกัน ให้รวมโค้ดทั้งหมดไว้ด้วยกัน

```rust
use part02::{SomethingOrNothing, Something, Nothing, vec_min};
```

ถ้าคุณแก้ไข `main.rs` ให้ใช้ part 03 แล้วรัน ตอนนี้มันควรจะถามให้คุณป้อนตัวเลข แล้วบอกค่าต่ำสุดให้ เห็นมั้ย ง่ายมากเลย

```rust
pub fn main() {
    let vec = read_vec();
    let min = vec_min(vec);
    min.print();
}
```

## แบบฝึกหัด

**แบบฝึกหัด 03.1a:** สร้าง trait `Print` ที่มี method `print` สำหรับพิมพ์ค่า แล้ว implement ให้กับ `i32`

<details>
<summary>เฉลย</summary>

```rust
pub trait Print {
    fn print(self);
}

impl Print for i32 {
    fn print(self) {
        println!("The number is: {}", self);
    }
}
```

</details>

**แบบฝึกหัด 03.1b:** สร้าง method `print2` สำหรับ `SomethingOrNothing<T>` ที่ใช้ trait `Print` โดยที่ `T` ต้อง implement `Print`

> **คำแนะนำ:** ใช้ syntax `impl<T: Print> SomethingOrNothing<T>` และเรียก `print` จาก `T` ภายใน

<details>
<summary>เฉลย</summary>

```rust
impl<T: Print> SomethingOrNothing<T> {
    fn print2(self) {
        match self {
            Nothing => println!("Nothing"),
            Something(t) => t.print(),
        }
    }
}
```

</details>

**แบบฝึกหัด 03.1c:** แก้ไข `main` ให้ใช้ `print2` แทน `print` และทดสอบกับ `Vec<i32>`

<details>
<summary>เฉลย</summary>

```rust
pub fn main() {
    let vec = read_vec();
    let min = vec_min(vec);
    min.print2(); // ใช้ print2 แทน print
}
```

</details>

**แบบฝึกหัด 03.2:** ต่อยอดจากแบบฝึกหัด 02.2 และ 03.1 ให้เขียนทุกอย่างที่จำเป็นสำหรับ `f32` เพื่อให้โปรแกรมของคุณทำงานกับจำนวนทศนิยมได้ (implement ทั้ง `Minimum` และ `Print` ให้ `f32`)

> **คำใบ้:** `f32` มี `Copy` เช่นเดียวกับ `i32` และมี method `min` ในตัวอยู่แล้ว

<details>
<summary>เฉลย</summary>

```rust
impl Minimum for f32 {
    fn min(self, b: Self) -> Self {
        self.min(b) // ใช้ method min ของ f32 โดยตรง
    }
}

impl Print for f32 {
    fn print(self) {
        println!("The float is: {}", self);
    }
}

fn main() {
    let vec = vec![3.14f32, 2.71, 1.41, 1.73];
    let min = vec_min(vec);
    min.print2(); // หรือ print() ถ้า implement ให้ SomethingOrNothing<f32> ด้วย
}
```

</details>

**แบบฝึกหัด 03.3 (ท้าทาย):** ปรับปรุง `read_vec` ให้คืนค่าเป็น `Result<Vec<i32>, String>` แทนที่จะพิมพ์ข้อผิดพลาดออกหน้าจอ และจัดการ error ใน `main` อย่างเหมาะสม (ไม่ใช้ `unwrap`)

<details>
<summary>เฉลย</summary>

```rust
fn read_vec_result() -> Result<Vec<i32>, String> {
    let mut vec = Vec::new();
    let stdin = io::stdin();
    
    for line in stdin.lock().lines() {
        let line = line.map_err(|e| format!("Read error: {}", e))?;
        match line.trim().parse::<i32>() {
            Ok(num) => vec.push(num),
            Err(e) => return Err(format!("'{}' is not a number: {}", line, e)),
        }
    }
    
    Ok(vec)
}

pub fn main() {
    match read_vec_result() {
        Ok(vec) => {
            let min = vec_min(vec);
            min.print();
        }
        Err(e) => println!("Error: {}", e),
    }
}
```

</details>
