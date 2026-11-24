---
title: "รันโค้ดสดบน Second Brain โดยไม่ต้องแคร์ Boilerplate"
slug: "รันโค้ดสดบน-second-brain"
date: "2026-11-24T14:00:00.000Z"
updated: "2026-11-24T14:00:00.000Z"
summary: "เมื่อ static second brain ต้องการ live playground ที่รัน React, Vue, Svelte, Solid ได้ทันทีจาก Markdown โดยไม่ต้องเขียนโค้ด mount และไม่แลกกับ performance budget"
tags:
  - "web"
  - "frontend"
  - "architecture"
  - "performance"
  - "thai"
para: "projects"
zettel: "permanent"
lang: "th"
hero: null
---

## บล็อก static ไม่จำเป็นต้องเป็นแค่ตัวหนังสือตายตัว

หนึ่งในหลักการสำคัญของ [[why-static-first]] คือ "Zero Blocking JS บน Class A Pages" เพื่อให้ผู้อ่านเปิดอ่านได้เร็วที่สุดในหลักไม่กี่สิบมิลลิวินาที ทว่าในฐานะนักพัฒนา เมื่อเราเขียนจดบันทึกทางเทคนิคใน [[ทำไมต้องเขียน-second-brain]] เรามักเจอกับตัวอย่างโค้ดที่อยากให้ผู้อ่าน (หรือตัวเราเองในอนาคต) กดรันและปรับแต่งดูผลลัพธ์ได้ทันที

แต่ปัญหาของ Code Playground ทั่วไปคือ:
1. **หนักเกินไป**: โหลดคอมไพเลอร์และเอดิเตอร์มาตั้งแต่หน้าแรก ทำให้เสีย Class A budget ทันที
2. **Boilerplate รกหูรกตา**: ตัวอย่างโค้ดสั้น ๆ แค่ปุ่มนับเลข แต่ต้องเขียน `import React`, `import { useState }`, `import { createRoot }` และหา DOM element มา `.render(...)` เกือบสิบบรรทัดจนกลบเนื้อหาสำคัญ

เราจึงสร้าง **In-Browser Interactive Runner** แบบ On-Demand ขึ้นมาบน panudet.dev

## ไม่โหลดสคริปต์จนกว่าจะกดใช้ (Zero-Cost until Click)

บนหน้าบทความนี้ ก่อนที่คุณจะกดปุ่มรัน **ไม่มี JavaScript ถูกโหลดเข้ามาเลยแม้แต่ไบต์เดียว** โค้ดทั้งหมดถูกไฮไลต์ด้วย Shiki ตั้งแต่ขั้นตอน Build Time (Prerendered)

แต่เมื่อผู้อ่านกดคลิก **Run** หรือกด **Inspect**:
- ระบบจะโหลด Web Component Runner แบบ Lazy Bundle (`/scripts/runner.js`) เข้ามาเพียงครั้งเดียว
- เปลี่ยนบล็อกโค้ดให้อยู่ใน CodeMirror พร้อม Syntax Highlighting, Folding และ Console Output
- ประมวลผลและสร้าง Sandbox Iframe ที่แยก Environment ออกจากตัวเว็บหลักอย่างปลอดภัย

```tsx live template=react
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div style={{ textAlign: 'center', padding: '1rem' }}>
      <p style={{ margin: '0 0 12px 0', fontSize: '1.1rem' }}>
        React 19 รันสดจาก Markdown ไร้ Boilerplate
      </p>
      <button
        style={{
          background: '#38bdf8',
          color: '#0f172a',
          fontWeight: 700,
          border: 'none',
          padding: '8px 18px',
          borderRadius: '6px',
          cursor: 'pointer'
        }}
        onClick={() => setCount((c) => c + 1)}
      >
        กดทดสอบ React Count: {count}
      </button>
    </div>
  );
}
```

## สั่ง template ได้จาก CommonMark Meta

เพียงแค่ใส่ `template=react`, `template=vue`, `template=svelte` หรือ `template=solid` ต่อท้าย code fence ตัวอย่างเช่น:

````markdown
```tsx live template=react
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>;
}
```
````

ระบบจะจัดการ:
- **Auto-Inject Hooks & Framework Core**: ดึง `useState`, `useEffect`, `createSignal`, `ref` มาให้โดยอัตโนมัติ
- **Auto-Mount**: ไม่ต้องสั่ง `createRoot().render()` หรือ `createApp().mount()` ตัว Runner ตรวจจับ Component หรือ Template Markup แล้วจัดวางลงใน DOM sandbox ให้เสร็จสรรพ
- **Support Vue Template เดี่ยวๆ**: สำหรับ Vue ถ้าเขียนเพียง `<button @click="count++">{{ count }}</button>` ระบบจะห่อเป็น Component และ reactive setup ให้ทันที

```html live template=vue
<div style="text-align: center; padding: 1rem;">
  <p style="margin: 0 0 12px 0; font-size: 1.1rem;">
    Vue 3 Single Template ไร้ setup() boilerplate
  </p>
  <button
    style="background: #42b883; color: #fff; font-weight: 700; border: none; padding: 8px 18px; borderRadius: 6px; cursor: pointer;"
    @click="count += 5"
  >
    Vue Count: {{ count }} (+5 ต่อคลิก)
  </button>
</div>
```

## TypeScript & External Packages (ESM)

ไม่ใช่แค่โค้ดเฟรมเวิร์กทั่วไป หากต้องการเรียกใช้งาน Library ภายนอกผ่าน npm ก็สามารถ `import` จาก ESM CDN อย่าง `esm.sh` ได้โดยตรง ตัวอย่างเช่นการคำนวณหรือการใช้งาน Lodash/Canvas:

```ts live
import confetti from 'https://esm.sh/canvas-confetti@1.9.4';

console.log("Runner พร้อมยิง Confetti!");

// สร้างปุ่มจำลอง UI สำหรับยิง Confetti สดใน Preview
const root = document.getElementById('root');
if (root) {
  root.innerHTML = `
    <div style="text-align: center; padding: 1.5rem 1rem;">
      <div style="font-size: 2.2rem; margin-bottom: 0.5rem;">🎉</div>
      <p style="margin: 0 0 1rem 0; font-weight: 500; font-size: 1.05rem;">
        Canvas Confetti รันสดจาก esm.sh
      </p>
      <button id="confetti-btn" style="
        background: #e06a4a;
        color: #fff;
        border: none;
        padding: 9px 20px;
        border-radius: 6px;
        font-weight: 600;
        cursor: pointer;
        font-family: inherit;
        box-shadow: 0 2px 8px rgba(224,106,74,0.3);
      ">
        ยิง Confetti อีกครั้ง! 🎊
      </button>
    </div>
  `;

  const btn = document.getElementById('confetti-btn');
  const fire = () => {
    confetti({
      particleCount: 90,
      spread: 70,
      origin: { y: 0.6 }
    });
  };

  btn?.addEventListener('click', fire);
  fire();
}
```

## ผ่านทุก Gate: สวยงาม รวดเร็ว และทดสอบ E2E อัตโนมัติ

ฟีเจอร์นี้ผ่านการพิสูจน์ด้วย Automated Playwright E2E Suite ครบทั้ง 8 หมวด พร้อมอัดวิดีโอหลักฐานการทดสอบแบบคมชัด และที่สำคัญที่สุดคือ:

- **Class A Performance Budget**: ยังคงรักษา 100/100 และขนาด HTML gzip ไม่เกินงบประมาณ
- **Native Security**: Iframe Sandbox ไม่อนุญาตให้สคริปต์ทดสอบเข้าถึง Cookie, LocalStorage หรือแทรกแซง Context ของ Second Brain
- **Seamless Design**: กลมกลืนกับ Theme และ Token ทั้งหมดของระบบ

นี่คือตัวอย่างของการผสานระหว่างความเร็วของ Static Site กับพลังของ Interactive Playground โดยไม่ทิ้งปรัชญาของความเรียบง่ายและรวดเร็ว
