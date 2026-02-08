## Part 11: Trait Objects, Box, Lifetime bounds

เราจะเล่นกับ closures อีกสักหน่อย ให้เรา implement กลไก "callback" แบบ generic บางอย่าง โดย provide สองฟังก์ชัน: การลงทะเบียน callback ใหม่ และการเรียก callbacks ทั้งหมดที่ลงทะเบียนไว้

ก่อนอื่น เราต้องหาวิธีเก็บ callbacks เห็นได้ชัดว่าจะมี `Vec` เกี่ยวข้อง เพื่อให้เราสามารถเติบโตจำนวน callbacks ที่ลงทะเบียนได้ตลอดเวลา callback จะเป็น closure คือสิ่งที่ implement `FnMut(i32)` (เราต้องการเรียกสิ่งนี้หลายครั้ง ดังนั้นแน่นอน `FnOnce` จะไม่ดี) ดังนั้นความพยายามแรกของเราอาจเป็นดังต่อไปนี้ ตอนนี้เราแค่ตัดสินใจว่า callbacks มี argument ของไทป์ `i32`

```rust
struct CallbacksV1<F: FnMut(i32)> {
    callbacks: Vec<F>,
}
```

อย่างไรก็ตาม สิ่งนี้จะไม่ทำงาน จำได้ไหมว่า "ไทป์" ของ closure เป็นเฉพาะเจาะจงต่อ environment ของตัวแปรที่ถูก capture closures ที่แตกต่างกันทั้งหมดที่ implement `FnMut(i32)` อาจมีไทป์ต่างกัน อย่างไรก็ตาม `Vec<F>` เป็น vector ที่มีไทป์สม่ำเสมอ

เราดังนั้นจะต้องการวิธีเก็บสิ่งต่างๆ ของไทป์ต่างกันในเวกเตอร์เดียวกัน เรารู้ว่าไทป์ทั้งหมดเหล่านี้ implement `FnMut(i32)` สำหรับสถานการณ์นี้ Rust provide **trait objects**: ความจริงคือ `FnMut(i32)` ไม่ได้เป็นแค่ trait มันยังเป็นไทป์ด้วย ที่สามารถให้กับอะไรก็ได้ที่ implement trait นี้ ดังนั้น เราอาจเขียนดังนี้

```rust
/* struct CallbacksV2 {
    callbacks: Vec<FnMut(i32)>,
} */
```

แต่ Rust บ่นเกี่ยวกับคำนิยามนี้ มันบอกอะไรบางอย่างเกี่ยวกับ "Sized" ปัญหาคืออะไร? ดูสิ สำหรับหลายสิ่งที่เราต้องการทำ มันเป็นสิ่งสำคัญที่ Rust รู้ขนาดที่แน่นอน คงที่ของไทป์ - คือ ไทป์นี้จะใหญ่แค่ไหนเมถูกแทนใน memory เช่น สำหรับ `Vec` elements ถูกเก็บติดกันทีละอัน จะเป็นไปได้อย่างไร โดยไม่มีขนาดคงที่? ประเด็นคือ `FnMut(i32)` อาจมีขนาดใดก็ได้ เราไม่รู้ว่า "ไทป์ที่ implement `FnMut(i32)`" นั้นใหญ่แค่ไหน Rust เรียกสิ่งนี้ว่า **unsized type** เมื่อใดก็ตามที่เราแนะนำตัวแปรไทป์ Rust จะเพิ่ม bound โดยอัตโนมัติให้กับตัวแปรนั้น เรียกร้องว่ามันมีขนาดคงที่ นั่นเป็นเหตุผลที่เราไม่ต้องกังวลเรื่องนี้มาก่อน

คุณสามารถ opt-out จาก bound โดยอัตโนมัตินี้โดยพูด `T: ?Sized` แล้ว `T` อาจมีหรือไม่มีขนาดคงที่ก็ได้

ดังนั้น เราจะทำอะไรได้ ถ้าเราไม่สามารถเก็บ callbacks ในเวกเตอร์ได้? เราสามารถใส่พวกมันใน **box** Semantically `Box<T>` คล้ายกับ `T` มาก: คุณเป็นเจ้าของข้อมูลที่เก็บไว้นั้นอย่างสมบูรณ์ บนเครื่องจริงๆ อย่างไรก็ตาม `Box<T>` เป็น pointer ไปยัง `T` ที่ถูก allocate บน heap มันคล้ายกับ `std::unique_ptr` ใน C++ ใน example ปัจจุบันของเรา สิ่งสำคัญคือเนื่องจากมันเป็น pointer `T` สามารถ unsized ได้ แต่ `Box<T>` เองจะมีขนาดคงที่เสมอ ดังนั้นเราสามารถใส่มันใน `Vec` ได้

```rust
pub struct Callbacks {
    callbacks: Vec<Box<FnMut(i32)>>,
}

impl Callbacks {
```

ทีนี้เราสามารถ provide ฟังก์ชันบางอย่าง constructor ควรเป็น

```rust
    pub fn new() -> Self {
        Callbacks { callbacks: Vec::new() }
    }
```

การลงทะเบียนแค่เก็บ callback

```rust
    pub fn register(&mut self, callback: Box<FnMut(i32)>) {
        self.callbacks.push(callback);
    }
```

เรายังสามารถเขียนเวอร์ชัน generic ของ `register` เช่นที่มันจะถูก instantiate ด้วย concrete closure type `F` บางอย่าง และทำการสร้าง `Box` และการแปลงจาก `F` ไปยัง `FnMut(i32)`

เพื่อให้สิ่งนี้ทำงานได้ เราต้องเรียกร้องว่าไทป์ `F` ไม่มี references ที่มีอายุสั้น ท้ายที่สุด เราจะเก็บมันในรายการ callbacks ของเราอย่างไม่มีกำหนด ถ้า closure มี pointer ไปยัง stackframe ของ caller ของเรา pointer นั้นอาจ invalid ตอนที่ closure ถูกเรียก เราสามารถบรรเทาสิ่งนี้โดยการ bound `F` ด้วย lifetime: `F: 'a` บอกว่าข้อมูลทั้งหมดของไทป์ `F` จะ outlive (คือ จะ valid อย่างน้อยนานเท่ากับ) lifetime `'a` ที่นี่ เราใช้ lifetime พิเศษ `'static` ซึ่งเป็น lifetime ของโปรแกรมทั้งหมด bound เดียวกันนี้ถูกเพิ่มโดยอัตโนมัติในเวอร์ชันของ `register` ข้างต้น และในการนิยามของ `Callbacks`

```rust
    pub fn register_generic<F: FnMut(i32) + 'static>(&mut self, callback: F) {
        self.callbacks.push(Box::new(callback));
    }
```

และที่นี่เราเรียก callbacks ทั้งหมดที่เก็บไว้

```rust
    pub fn call(&mut self, val: i32) {
```

เนื่องจากพวกมันเป็นไทป์ `FnMut` เราต้อง mutably borrow พวกมัน

```rust
        for callback in self.callbacks.iter_mut() {
```

ที่นี่ `callback` มีไทป์ `&mut Box<FnMut(i32)>` เราสามารถใช้ประโยชน์จากความจริงที่ว่า `Box` เป็น smart pointer: โดยเฉพาะ เราสามารถใช้มันเสมือนมันเป็น normal reference และใช้ `*` เพื่อไปยัง contents ของมัน แล้วเราได้รับ mutable reference ไปยัง contents เหล่านี้ เพราะเราเรียก `FnMut`

```rust
            (&mut *callback)(val);
```

เหมือนกับกรณีของ normal references สิ่งนี้โดยทั่วไปเกิดขึ้นโดยอัตโนมัติกับ smart pointers ดังนั้นเราสามารถเรียกฟังก์ชันโดยตรงได้ ลองลบ `&mut *` ออก

ความแตกต่างกับ reference คือ `Box` หมายถึง full ownership: เมื่อคุณ drop box (คือ เมื่อ instance ทั้งหมดของ `Callbacks` ถูก drop) content ที่มันชี้ไปบน heap จะถูก drop ด้วย

```rust
        }
    }
}
```

ทีนี้เราพร้อมสำหรับ demo อย่าลืมแก้ไข `main.rs` เพื่อรันโค้ดนี้

```rust
pub fn main() {
    let mut c = Callbacks::new();
    c.register(Box::new(|val| println!("Callback 1: {}", val)));
    c.call(0);
    {
```

เรายังสามารถลงทะเบียน callbacks ที่ modify environment ของพวกมันได้ โดยค่าเริ่มต้น Rust จะพยายาม capture reference ไปยัง `count` เพื่อยืมมัน อย่างไรก็ตาม สิ่งนั้นไม่ทำงานครั้งนี้ จำ bound `'static` ข้างต้นได้ไหม? การยืม `count` ใน environment จะ violate bound นั้น เนื่องจาก reference valid สำหรับ block นี้เท่านั้น ถ้า callbacks ถูก trigger ภายหลัง เราจะมีปัญหา เราต้องบอก Rust อย่างชัดเจนให้ move ownership ของตัวแปรเข้าไปใน closure Environment ของมันทีนี้จะมี `usize` มากกว่า `&mut usize` และ closure ไม่มีผลกระทบต่อตัวแปร local นี้

```rust
        let mut count: usize = 0;
        c.register_generic(move |val| {
            count = count + 1;
            println!("Callback 2: {} ({}. time)", val, count);
        });
    }
    c.call(1);
    c.call(2);
}
```

## Run-time behavior

เมื่อคุณรันโปรแกรมข้างต้น Rust รู้ได้อย่างไรว่าต้องทำอะไรกับ callbacks? เนื่องจาก unsized type ขาดข้อมูลบางอย่าง pointer ไปยังไทป์เช่นนี้ (ไม่ว่าจะเป็น `Box` หรือ reference) จะต้อง complete ข้อมูลนี้ เราบอกว่า pointers ไปยัง trait objects เป็น **fat** พวกมันเก็บไม่เพียงแค่ address ของ object แต่ (ในกรณีของ trait objects) ยังมี **vtable**: ตารางของ function pointers กำหนดโค้ดที่รันเมื่อ trait method ถูกเรียก มีข้อจำกัดบางอย่างสำหรับ traits ที่จะถูกใช้เป็น trait objects สิ่งนี้เรียกว่า **object safety** และอธิบายในเอกสารและ reference ในกรณีของ trait `FnMut` มีเพียง action เดียวที่ต้องทำ: การเรียก closure คุณดังนั้นสามารถคิดว่า pointer ไปยัง `FnMut` เป็น pointer ไปยังโค้ด และ pointer ไปยัง environment นี่คือวิธีที่ Rust recover encoding ทั่วไปของ closures เป็นกรณีพิเศษของแนวคิดที่ทั่วไปกว่า

เมื่อใดก็ตามที่คุณเขียน generic function คุณมีตัวเลือก: คุณสามารถทำให้มัน generic เหมือน `register_generic` หรือคุณสามารถใช้ trait objects เหมือน `register` อันหลังจะส่งผลให้มีเพียง single compiled version (มากกว่าหนึ่งเวอร์ชันต่อไทป์ที่มันถูก instantiate ด้วย) สิ่งนี้ทำให้โค้ดเล็กลง แต่คุณจ่ายค่าใช้จ่ายของ virtual function calls (แน่นอน ในกรณีของ `register` ข้างต้น ไม่มีฟังก์ชันถูกเรียกบน trait object) ไม่สวยงามหรือที่ traits สามารถจัดการ tradeoff นี้ได้อย่างดี (และมากกว่านั้น อย่างที่เราเห็น เช่น closures และ operator overloading)?

**แบบฝึกหัด 11.1:** เราทำตัวเลือกแบบ arbitrary ของการใช้ `i32` สำหรับ arguments ทำให้โครงสร้างข้อมูลข้างต้น generic เพื่อทำงานกับไทป์ `T` ใดๆ ที่ถูกส่งไปยัง callbacks เนื่องจากคุณต้องเรียก callbacks หลายตัวด้วย `val: T` เดียวกัน (ในฟังก์ชัน `call` ของเรา) คุณจะต้องจำกัด `T` ให้เป็น `Copy` types หรือส่ง reference
