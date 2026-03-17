# Rust-101 (ฉบับภาษาไทย)

[![GitHub license](https://img.shields.io/github/license/pharmacist-sabot/rust-101-th)](https://github.com/pharmacist-sabot/rust-101-th/blob/main/LICENSE)
[![Built with mdBook](https://img.shields.io/badge/Built%20with-mdBook-blue)](https://rust-lang.github.io/mdBook/)

## เกี่ยวกับโปรเจกต์

โปรเจกต์นี้คือการแปลและเรียบเรียงเนื้อหาจากหนังสือ "Rust-101" โดย Ralf Jung ให้เป็นภาษาไทย เพื่อให้ผู้ที่สนใจเรียนรู้ภาษา Rust ในประเทศไทยสามารถเข้าถึงและทำความเข้าใจเนื้อหาได้อย่างสะดวกยิ่งขึ้น โดยมีเป้าหมายที่จะเป็นแหล่งเรียนรู้พื้นฐานที่ครอบคลุมและเข้าใจง่ายสำหรับนักพัฒนาชาวไทย

## ต้นฉบับและใบอนุญาต

เนื้อหาในโปรเจกต์นี้แปลและเรียบเรียงจาก:

* **ชื่อหนังสือ:** Rust-101
* **ผู้เขียน:** Ralf Jung
* **ต้นฉบับ:** [https://www.ralfj.de/projects/rust-101/](https://www.ralfj.de/projects/rust-101/)
* **ใบอนุญาตต้นฉบับ:** Creative Commons Attribution–ShareAlike 4.0 International (CC BY-SA 4.0)

งานแปลนี้เผยแพร่ภายใต้ใบอนุญาต **[Creative Commons Attribution–ShareAlike 4.0 International (CC BY-SA 4.0)](./LICENSE)** เช่นกันครับ ซึ่งหมายความว่าคุณสามารถนำไปใช้งาน ดัดแปลง และเผยแพร่ต่อได้ โดยต้องให้เครดิตแก่ผู้สร้างต้นฉบับและเผยแพร่งานดัดแปลงภายใต้ใบอนุญาตเดียวกัน

## วิธีการอ่าน/สร้างหนังสือ

โปรเจกต์นี้ใช้ `mdBook` ในการสร้างหนังสือ หากคุณต้องการอ่านหนังสือในรูปแบบเว็บ หรือสร้างหนังสือด้วยตัวเอง สามารถทำตามขั้นตอนด้านล่างนี้ได้เลยครับ

### ข้อกำหนดเบื้องต้น

คุณต้องติดตั้ง `mdBook` ก่อน โดยสามารถติดตั้งได้ด้วยคำสั่ง:

```bash
cargo install mdbook
```

### การสร้างหนังสือ

หลังจากติดตั้ง `mdBook` แล้ว ให้โคลนโปรเจกต์นี้และใช้คำสั่ง `mdbook build`:

```bash
git clone https://github.com/pharmacist-sabot/rust-101-th.git
cd rust-101-th
mdbook build
```

คำสั่งนี้จะสร้างไฟล์ HTML ของหนังสือไว้ในโฟลเดอร์ `./book`

### การอ่านหนังสือในเบราว์เซอร์

คุณสามารถเปิดไฟล์ `index.html` ในโฟลเดอร์ `./book` ด้วยเบราว์เซอร์ของคุณโดยตรง หรือใช้คำสั่ง `mdbook serve` เพื่อรันเซิร์ฟเวอร์พัฒนา:

```bash
mdbook serve
```

จากนั้นเปิดเบราว์เซอร์ไปที่ `http://localhost:3000` (หรือพอร์ตอื่นที่ระบุ) เพื่อดูหนังสือ

## การมีส่วนร่วม (Contributing)

เรายินดีต้อนรับทุกท่านที่มีความสนใจในการปรับปรุงงานแปลนี้ ไม่ว่าจะเป็นการแก้ไขคำผิด, ปรับปรุงสำนวน, หรือเสนอแนะเนื้อหาเพิ่มเติม

1. Fork repository นี้
2. สร้าง branch ใหม่สำหรับงานของคุณ (`git checkout -b feature/your-feature-name`)
3. ทำการเปลี่ยนแปลงและ commit (`git commit -m 'Add some feature'`)
4. Push ไปยัง branch ของคุณ (`git push origin feature/your-feature-name`)
5. สร้าง Pull Request

## ติดต่อ

หากมีข้อสงสัยหรือข้อเสนอแนะ สามารถติดต่อได้ที่ [pharmacistsabot@gmail.com]
