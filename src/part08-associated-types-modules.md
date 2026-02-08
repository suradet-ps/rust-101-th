## Part 08: Associated Types, Modules

```rust
use std::{cmp, ops};
use part05::BigInt;
```

เป้าหมายถัดไปของเรา ให้เรา implement การบวกสำหรับ `BigInt` ของเรา ปัญหาหลักที่นี่คือการจัดการกับ overflow ก่อนอื่น เราจะต้องตรวจจับเมื่อ overflow เกิดขึ้น สิ่งนี้ถูกเก็บไว้ใน bit ที่เรียกว่า carry และเราต้อง carry ข้อมูลนี้ไปยังคู่ของหลักถัดไปที่เราบวก primitive หลักของการบวกดังนั้นคือการบวกสองหลักและ carry แล้วคืนหลักผลบวกและ carry ถัดไป

ดังนั้น ให้เราเขียนฟังก์ชัน "add with carry" และให้ไทป์ที่เหมาะสมกับมัน สังเกตการสนับสนุน native ของ Rust สำหรับ pairs

```rust
fn overflowing_add(a: u64, b: u64, carry: bool) -> (u64, bool) {
```

ท่าทีของ Rust ต่อ integer overflows อาจ surprise บ้าง: โดยทั่วไป เมื่อเราเขียน `a + b` overflow ถือว่าเป็น error ถ้าคุณ compile โปรแกรมใน debug mode Rust จะตรวจสอบ error นั้นจริงๆ และแพนิกโปรแกรมในกรณีของ overflows เพื่อเหตุผลด้านประสิทธิภาพ ไม่มีการตรวจสอบแบบนี้ถูกแทรกสำหรับ release builds เหตุผลสำหรับสิ่งนี้คือ vulnerabilities ด้านความปลอดภัยที่ร้ายแรงหลายอย่างเกิดจาก integer overflows ดังนั้นการสมมติ "ตามค่าเริ่มต้น" ว่าพวกมันเป็นที่ตั้งใจไว้นั้นอันตราย

ถ้าคุณต้องการให้ overflow เกิดขึ้นอย่างชัดเจน คุณสามารถเรียกฟังก์ชัน `wrapping_add` (ดูเอกสาร มีฟังก์ชันคล้ายกันสำหรับการดำเนินการ arithmetic อื่นๆ) ยังมีฟังก์ชันคล้ายกัน `checked_add` ฯลฯ เพื่อบังคับการตรวจสอบ overflow

```rust
    let sum = a.wrapping_add(b);
```

ถ้า overflow เกิดขึ้น แล้ว sum จะเล็กกว่า summands ทั้งสอง โดยไม่มี overflow แน่นอน มันจะมีขนาดใหญ่พอๆ กับทั้งสองอย่างน้อย ดังนั้น ให้เราเลือกอันหนึ่งและตรวจสอบ

```rust
    if sum >= a {
```

การบวกไม่ได้ overflow

**แบบฝึกหัด 08.1:** เขียนโค้ดเพื่อจัดการการบวก carry ในกรณีนี้

```rust
        unimplemented!()
    } else {
```

มิฉะนั้น การบวก overflow มันเป็นไปไม่ได้สำหรับการบวกของ carry ที่จะ overflow อีกครั้ง เพราะเราแค่บวก 0 หรือ 1

```rust
        (sum + if carry { 1 } else { 0 }, true)
    }
}
```

`overflow_add` เป็นฟังก์ชันที่ซับซ้อนพอสมควรที่ testcase มีเหตุผล สิ่งนี้ควรช่วยคุณตรวจสอบคำตอบของแบบฝึกหัดด้วย

```rust
/*#[test]*/
fn test_overflowing_add() {
    assert_eq!(overflowing_add(10, 100, false), (110, false));
    assert_eq!(overflowing_add(10, 100, true), (111, false));
    assert_eq!(overflowing_add(1 << 63, 1 << 63, false), (0, true));
    assert_eq!(overflowing_add(1 << 63, 1 << 63, true), (1, true));
    assert_eq!(overflowing_add(1 << 63, (1 << 63) - 1, true), (0, true));
}
```

## Associated Types

ทีนี้เราพร้อมที่จะเขียนฟังก์ชันการบวกสำหรับ `BigInt` อย่างที่คุณอาจเดาได้ ตัวดำเนินการ `+` เชื่อมต่อกับ trait (`std::ops::Add`) ซึ่งเรากำลังจะ implement สำหรับ `BigInt`

โดยทั่วไป การบวกไม่จำเป็นต้องเป็น homogeneous: คุณอาจบวกสิ่งต่างชนิดกัน เช่น vectors และ points ดังนั้นเมื่อ implement `Add` สำหรับไทป์หนึ่ง เราต้องระบุไทป์ของ operand อีกอัน ในกรณีนี้ มันจะเป็น `BigInt` เช่นกัน (และเราอาจละมันไปได้ เพราะนั่นคือค่าเริ่มต้น)

```rust
impl ops::Add<BigInt> for BigInt {
```

นอกจาก static functions และ methods แล้ว traits สามารถมี **associated types** ได้: นี่คือไทป์ที่ถูกเลือกโดยทุกการ implement เฉพาะของ trait methods ของ trait สามารถอ้างถึงไทป์นั้นได้ ในกรณีของการบวก มันถูกใช้เพื่อให้ไทป์ของผลลัพธ์ (ดูเอกสารของ `Add` ด้วย)

โดยทั่วไป คุณสามารถถือว่า `BigInt` สองตัวข้างต้น (ในบรรทัด impl) เป็น input types ของการค้นหา trait: เมื่อ `a + b` ถูก invoke โดยที่ `a` มีไทป์ `T` และ `b` มีไทป์ `U` Rust พยายามหา implementation ของ `Add` สำหรับ `T` ที่ไทป์ขวาคือ `U` associated types จากอีกด้านหนึ่ง เป็น output types: สำหรับทุก combination ของ input types มี result type เฉพาะที่ถูกเลือกโดยการ implement `Add` ที่สอดคล้องกัน

ที่นี่ เราเลือก result type ให้เป็น `BigInt` อีกครั้ง

```rust
    type Output = BigInt;
```

ทีนี้เราสามารถเขียนฟังก์ชันจริงที่ทำการบวกได้

```rust
    fn add(self, rhs: BigInt) -> Self::Output {
```

เรารู้ว่าผลลัพธ์จะมีความยาวอย่างน้อยเท่ากับ operand ที่ยาวกว่าสองตัว ดังนั้นเราสามารถสร้างเวกเตอร์ที่มี capacity เพียงพอเพื่อหลีกเลี่ยง reallocations ที่แพง

```rust
        let max_len = cmp::max(self.data.len(), rhs.data.len());
        let mut result_vec: Vec<u64> = Vec::with_capacity(max_len);
        let mut carry = false; /* the current carry bit */
        
        for i in 0..max_len {
            let lhs_val = if i < self.data.len() { self.data[i] } else { 0 };
            let rhs_val = if i < rhs.data.len() { rhs.data[i] } else { 0 };
```

คำนวณหลักถัดไปและ carry แล้ว เก็บหลักสำหรับผลลัพธ์ และ carry สำหรับภายหลัง สังเกตว่าเราสามารถได้ชื่อสำหรับสอง components ของ pair ที่ `overflowing_add` คืนมา

```rust
            let (sum, new_carry) = overflowing_add(lhs_val, rhs_val, carry);
            result_vec.push(sum);
            carry = new_carry;
        }
```

**แบบฝึกหัด 08.2:** จัดการ carry สุดท้าย และคืนผลบวก

```rust
        unimplemented!()
    }
}
```

## Traits และ reference types

ถ้าคุณตรวจสอบฟังก์ชันการบวกข้างต้นอย่างใกล้ชิด คุณจะสังเกตว่ามันจริงๆ แล้ว consume ownership ของ operands ทั้งสองเพื่อสร้างผลลัพธ์ นี่ไม่ใช่สิ่งที่เราต้องการโดยทั่วไป เราอยากสามารถบวกสอง `&BigInt` ได้

การเขียนสิ่งนี้ออกมาจะยุ่งยากหน่อย เพราะ trait implementations (ไม่เหมือนกับ functions) ต้องการการ annotate lifetimes อย่างชัดเจนเต็มรูปแบบ ตรวจสอบให้แน่ใจว่าคุณเข้าใจคำนิยามต่อไปนี้อย่างแน่นอน สังเกตว่าเราสามารถ implement trait สำหรับ reference type ได้!

```rust
impl<'a, 'b> ops::Add<&'a BigInt> for &'b BigInt {
    type Output = BigInt;
    
    fn add(self, rhs: &'a BigInt) -> Self::Output {
```

**แบบฝึกหัด 08.3:** Implement ฟังก์ชันนี้

```rust
        unimplemented!()
    }
}
```

**แบบฝึกหัด 08.4:** Implement สอง combination ที่ขาดหายไปของ arguments สำหรับ `Add` คุณไม่ควรต้อง duplicate implementation

## Modules

อย่างที่คุณเรียนรู้ tests สามารถเขียนได้ตรงกลางการพัฒนาของคุณใน Rust อย่างไรก็ตาม ถือว่าเป็น style ที่ดีที่จะรวม tests ทั้งหมดเข้าด้วยกัน สิ่งนี้มีประโยชน์โดยเฉพาะในกรณีที่คุณเขียน utility functions สำหรับ tests ที่โค้ดอื่นไม่ควรใช้

Rust เรียกกลุ่มของ definitions ที่ถูกจัดกลุ่มเข้าด้วยกันว่า **module** คุณสามารถใส่ tests ใน submodule ได้ดังนี้ attribute `cfg` ควบคุมว่า module นี้จะถูก compile หรือไม่: ถ้าเราเพิ่ม functions ที่มีประโยชน์สำหรับ testing Rust จะไม่ยุ่งยาก compile พวกมันเมื่อคุณแค่ build โปรแกรมของคุณสำหรับการใช้งานปกติ นอกจากนั้น tests ทำงานเหมือนเดิม

```rust
#[cfg(test)]
mod tests {
    use part05::BigInt;
    
    /*#[test]*/
    fn test_add() {
        let b1 = BigInt::new(1 << 32);
        let b2 = BigInt::from_vec(vec![0, 1]);
        assert_eq!(&b1 + &b2, BigInt::from_vec(vec![1 << 32, 1]));
    }
```

**แบบฝึกหัด 08.5:** เพิ่ม cases อีกสักหน่อยให้กับ test นี้

```rust
}
```

อย่างที่กล่าวไปแล้ว นอก module เฉพาะ items ที่ประกาศเป็น public ด้วย `pub` เท่านั้นที่อาจถูกใช้ Submodules สามารถเข้าถึงทุกอย่างที่นิยามใน parents ของพวกมัน Modules เองก็ซ่อนจากภายนอกตามค่าเริ่มต้นเช่นกัน และสามารถทำให้เป็น public ด้วย `pub` เมื่อคุณใช้ identifier (หรือ ทั่วไปกว่านั้น path เช่น `mod1::submod::name`) มันถูกตีความว่าเป็น relative ต่อ module ปัจจุบัน ดังนั้น เช่น เพื่อเข้าถึง `overflowing_add` จากภายใน `my_mod` คุณต้องให้ path ที่ชัดเจนกว่าโดยเขียน `super::overflowing_add` ซึ่งบอก Rust ให้ดูใน parent module

คุณสามารถทำให้ชื่อจาก modules อื่นๆ พร้อมใช้งาน locally ด้วย `use` ตามค่าเริ่มต้น `use` ทำงาน globally ดังนั้น เช่น `use std;` นำเข้าชื่อ global `std` โดยการเพิ่ม `super::` หรือ `self::` ที่จุดเริ่มต้นของ path คุณทำให้มันเป็น relative ต่อ parent หรือ module ปัจจุบันตามลำดับ (คุณยังสามารถสร้าง absolute path อย่างชัดเจนโดยเริ่มด้วย `::` เช่น `::std::cmp::min`) คุณสามารถพูด `pub use path;` เพื่อนำเข้าชื่อและทำให้พวกมัน publicly พร้อมใช้งานสำหรับคนอื่นๆ พร้อมกัน สุดท้าย คุณสามารถนำเข้า public items ทั้งหมดของ module ในครั้งเดียวด้วย `use module::*;`

Modules สามารถใส่ลงในไฟล์แยกต่างหากด้วย syntax `mod name;` เพื่ออธิบายสิ่งนี้ ให้ผมออกทางเล็กน้อยผ่านกระบวนการ compile ของ Rust Cargo เริ่มต้นด้วยการ invoke `rustc` บนไฟล์ `src/lib.rs` หรือ `src/main.rs` ขึ้นอยู่กับว่าคุณ compile application หรือ library เมื่อ `rustc` พบ `mod name;` มันจะมองหาไฟล์ `name.rs` และ `name/mod.rs` และไป compile ต่อที่นั่น (มันเป็น error ถ้าทั้งสองมีอยู่) คุณสามารถคิดว่า contents ของไฟล์ถูกฝังไว้ที่ตำแหน่งนี้ อย่างไรก็ตาม เฉพาะไฟล์ที่การ compile เริ่มต้น และไฟล์ `name/mod.rs` เท่านั้นที่สามารถ load modules จากไฟล์อื่นๆ ได้ สิ่งนี้รับประกันว่า directory structure สะท้อน structure ของ modules โดยที่ `mod.rs` `lib.rs` และ `main.rs` แทน directory หรือ crate ตัวมันเอง (คล้ายกับ เช่น `__init__.py` ใน Python)

**แบบฝึกหัด 08.6:** เขียนฟังก์ชันการลบ และ testcases สำหรับมัน ตัดสินใจเองว่าคุณอยากจัดการผลลัพธ์ติดลบอย่างไร เช่น คุณอาจอยากคืน `Option` จะ panic หรือคืน 0
