---
title: "Mini LocalQR Rowcall Sys"
date: 2026-08-19
description: "How I solve row call on paper problem by using technology."
tags: [react, express, javascript]
---

# How I Built a QR Roll Call System to Fix a Problem I Hated Every Class
> Scroll for မြန်မာဘာသာဖြင့် (Myanmar Version)
*By Aster Julian Ray · Senior Mentor, UCS-Pyay · @picakhant*

Last week I stood in front of my class at the **University of Computer Studies, Pyay (UCS-Pyay)**,
teaching the students how to build a **React + Express CRUD application**. That part was great.
But at the end of the class came the part I dreaded: **roll call**.

I shouted each name, waited for a "here!", ticked a mark on paper, and repeated that for every
student. Then, after class, I'd retype everything into a report. It was slow, noisy, tiring, and
it ate into time I'd rather spend teaching.

That night I decided: **I'm a developer. I can fix this.**

---

## The Idea

What if each student had a **QR code** that encoded their student ID, and I could scan all of
them with my laptop's webcam in under a minute? The system would:

1. read each QR as it flashes in front of the camera,
2. ignore duplicates,
3. compare against the full class list,
4. save a clean, dated attendance report — and show me exactly who's missing.

No extra hardware. No internet needed. No database to configure. Just my laptop, a webcam,
and a printed piece of paper per student.

That became **RCsys**.

![CS-001](assets/qr/CS-001_Card.png)
![CS-002](assets/qr/CS-002_Card.png)
![CS-003](assets/qr/CS-003_Card.png)

> These are the actual QR cards the system generates — one per student.

---

## The Tech I Chose

### Frontend — React + Vite + Tailwind

The class was literally learning React and Express that day, so this was a natural fit.

- **Vite** for a fast, modern dev experience.
- **React Router** for two routes: `/` (scanner) and `/admin` (QR card generator).
- **Tailwind CSS** — I went with a **dark "hacker terminal" theme** because, well, I'm a CS
  mentor and it looks awesome. The UI reads like a terminal: `root@404SNF:~# RCsys`,
  `[ EXECUTE_SAVE ]`, `SYSTEM_LOGS`, `>_` prompts. It makes the tool feel like a tiny piece
  of software a developer built for themselves.

Two libraries make the magic happen:

- **[qrcode.react](https://github.com/zpao/qrcode.react)** — renders a QR canvas per student
  on the admin page. Each card encodes the student's ID (e.g. `CS-001`), drawn on a white
  background with the ID and name underneath so it's print-friendly.
- **[@yudiel/react-qr-scanner](https://github.com/yudielcurbelo/react-qr-scanner)** — opens the
  webcam and decodes QR codes in real time. It's what turns my camera into a roll-call machine.

Downloading cards is handled with **JSZip + FileSaver** — one click gives me a ZIP with every
student's card, ready to print.

### Backend — Express 5, no database

The backend is deliberately minimal: an **Express** server that reads and writes plain text
files under `~/RCsys/`.

```
~/RCsys/
├── student/list.txt            # "CS-001, Aung Aung" — one line per student
└── daily/JS_Section_A_2026-07-05.txt   # dated attendance reports
```

Two endpoints:

- `GET /api/students` — returns the class list.
- `POST /api/attendance` — receives the scanned IDs + session name, computes present/absent,
  and writes a human-readable report.

No MongoDB, no Postgres, no ORM. For a single classroom tool that runs on my own machine, a
text file is more reliable, simpler to back up, and easier to explain to students than a
full database stack. Sometimes the simplest tool is the right tool.

The server also serves the built React app, so the whole system is one process on
**http://localhost:5000**.

---

## Building It — Step by Step

### 1. Student list first

The data starts as a plain text file — `~/RCsys/student/list.txt`. The server auto-creates
the folders on first run and even seeds a dummy list so you can test before entering real names.

### 2. The Admin page (QR generator)

Fetch the list from the API, map over the students, and render a QR canvas for each one:

```jsx
<QRCodeCanvas id={`qr-${student.id}`} value={student.id} size={130} />
```

Each card is composited onto a white canvas with the ID and name below, then exported as PNG —
individually or packed into one ZIP.

### 3. The Scanner page (the fun part)

This is the heart of the app. A `Scanner` component decodes codes continuously. My scan handler:

- trims the decoded value,
- checks a `Set` so the same student can't be counted twice (people always scan twice by accident!),
- plays a short **beep** so both me and the student get instant confirmation,
- appends the ID to the on-screen `SYSTEM_LOGS`.

```jsx
if (!scannedSet.current.has(studentId)) {
  scannedSet.current.add(studentId);
  playBeep();
  setScannedIds(Array.from(scannedSet.current));
}
```

### 4. Saving the roll call

When the scanning is done, I hit `[ EXECUTE_SAVE ]`. A modal asks for a **session name**
(e.g. `JS_Section_A`), and the frontend POSTs the present IDs. The server:

- marks everyone in the list who isn't in the scanned set as **absent**,
- writes a dated report to `~/RCsys/daily/`,
- returns the result.

The UI then shows a summary with the full **missing list** — so I immediately know who to chase
up, before the students even leave the room.

---

## Challenges I Hit Along the Way

1. **Duplicate scans** — The same card inevitably gets scanned twice. Solved with a `Set` and
   instant beep feedback.

2. **Report filenames** — Session names contain spaces and (in my case) Burmese characters.
   I sanitize the name to `[A-Za-z0-9_\u1000-\u109F]` so filenames stay safe while still
   supporting Myanmar text.

3. **Client-side routing on the server** — React Router needs the server to fall back to
   `index.html` for any non-API route, otherwise refreshing `/admin` 404s:

   ```js
   app.get(/.*/, (req, res) => res.sendFile(path.join(distPath, "index.html")));
   ```

4. **CORS between dev servers** — In development the Vite dev server (:5173) talks to the
   Express API (:5000), so the backend enables `cors()`.

5. **The beep** — I used the Web Audio API to synthesize a quick sine beep instead of shipping
   an audio file. Tiny touch, but it makes scanning feel responsive.

---

## The Result

Roll call went from **5+ minutes of shouting** to **about 30 seconds of scanning**.

- Students love the "high-tech" card scanning — it feels like boarding a plane.
- I get an accurate present/absent list with zero manual tallying.
- Reports are saved as dated text files, ready to paste into records.
- No database, no internet, no deployment — it just runs on my laptop.

---

## What I Learned

- **Build tools for your own friction.** The pain I felt at the end of every class became the
  perfect spec for this project.
- **A QR code is just a string.** Once I realized the code can encode any ID, the whole system
  is really just *string → camera → lookup → save*.
- **Simple storage wins.** My instinct was to add a database; the honest answer was a text file.
- **Good UX is small details.** The beep, the dedup, the terminal aesthetic, the ZIP download —
  none were "required," but together they make the tool feel great to use.
- **Teach by example.** I used the exact same stack (React + Express) I was mentoring that week —
  now my students have a real, working example of the CRUD concepts I explained on the board.

---

## Try It

The project is open — grab the code, add your own `~/RCsys/student/list.txt`, and take your
next roll call in seconds.

- **Author:** Aster Julian Ray — Senior Mentor, UCS-Pyay
- **GitHub:** [@picakhant](https://github.com/picakhant)

*No more shouting roll call. Ever.*

---

## မြန်မာဘာသာဖြင့် (Myanmar Version)

# အတန်းတိုင်းမှာ ကျွန်တော်အမုန်းဆုံး ပြဿနာတစ်ခုကို ဖြေရှင်းဖို့ QR Roll Call System တစ်ခု ဘယ်လိုတည်ဆောက်ခဲ့လဲ

*By Aster Julian Ray · Senior Mentor, UCS-Pyay · @picakhant*

ပြီးခဲ့တဲ့အပတ်က ပြည်ကွန်ပျူတာတက္ကသိုလ် (UCS-Pyay) မှာ ကျောင်းသားတွေကို **React + Express CRUD application** ဘယ်လိုရေးရမလဲဆိုတာ သင်ပေးနေခဲ့ပါတယ်။ စာသင်ရတဲ့အပိုင်းက အရမ်းကောင်းပါတယ်။ ဒါပေမဲ့ အတန်းပြီးသွားတဲ့အချိန်မှာ ကျွန်တော်အမုန်းဆုံးအချိန် ရောက်လာပါတော့တယ်။ အဲဒါကတော့ **Roll call (နာမည်ခေါ်ခြင်း)** ပါပဲ။

နာမည်တစ်ယောက်ချင်းစီကို အော်ခေါ်ရတယ်၊ "ရှိပါတယ်" ဆိုတဲ့အသံကို စောင့်ရတယ်၊ စာရွက်ပေါ်မှာ အမှန်ခြစ်ရတယ်၊ ကျောင်းသားတိုင်းအတွက် ဒါကို ထပ်ခါတလဲလဲ လုပ်ရတယ်။ ပြီးတော့ အတန်းပြီးသွားရင် အဲဒီစာရင်းကို Report အနေနဲ့ ကွန်ပျူတာထဲ ပြန်ရိုက်ထည့်ရတယ်။ အချိန်အရမ်းကုန်တယ်၊ ဆူညံတယ်၊ ပင်ပန်းတယ်၊ ပြီးတော့ စာသင်ဖို့သုံးရမယ့် အချိန်တွေကိုပါ ဝါးမျိုသွားတယ်။ 

အဲဒီညမှာပဲ ကျွန်တော် ဆုံးဖြတ်လိုက်တယ် - **"ငါက Developer တစ်ယောက်ပဲ။ ဒီပြဿနာကို ငါဖြေရှင်းနိုင်ရမယ်"** လို့။

---

### အိုင်ဒီယာ

ကျောင်းသားတစ်ယောက်ချင်းစီဆီမှာ သူတို့ရဲ့ Student ID ကို ထည့်သွင်းထားတဲ့ **QR code** လေးတွေ ရှိနေပြီး၊ အဲဒါတွေကို ကျွန်တော့် Laptop ရဲ့ Webcam ကနေတစ်ဆင့် တစ်မိနစ်အတွင်း အကုန်လုံးကို ဖတ်လိုက်လို့ရရင် ဘယ်လောက်ကောင်းမလဲ? ဒီစနစ်လေးက -

1. ကင်မရာရှေ့ရောက်လာတဲ့ QR တိုင်းကို ဖတ်မယ်။
2. နှစ်ခါထပ်ဖတ်မိတဲ့ အမှားတွေကို ပယ်မယ်။
3. အတန်းစာရင်းတစ်ခုလုံးနဲ့ တိုက်စစ်မယ်။
4. ဘယ်သူတွေ ပျက်လဲဆိုတာ အတိအကျပြပေးပြီး သပ်ရပ်တဲ့ Attendance Report တစ်ခုကို မှတ်တမ်းတင်ပေးမယ်။

ဘာ Hardware အပိုမှ မလိုဘူး။ အင်တာနက် မလိုဘူး။ Database တွေ Setup လုပ်နေစရာ မလိုဘူး။ Laptop တစ်လုံး၊ Webcam တစ်ခုနဲ့ ကျောင်းသားတစ်ယောက်အတွက် စာရွက်တစ်ရွက်စီပဲ လိုတယ်။

ဒါကတော့ **RCsys** ဖြစ်လာမယ့် အစပါပဲ။

> *(အထက်ပါ ပုံများမှာ စနစ်မှ ထုတ်ပေးထားသော ကျောင်းသားတစ်ဦးချင်းစီအတွက် အသုံးပြုမည့် QR ကတ်အစစ်များ ဖြစ်ပါသည်။)*

---

### ကျွန်တော်ရွေးချယ်ခဲ့တဲ့ နည်းပညာများ (The Tech I Chose)

#### Frontend — React + Vite + Tailwind

အဲဒီနေ့က ကျောင်းသားတွေကို React နဲ့ Express အကြောင်း သင်ပေးနေတာဆိုတော့ ဒီ Stack ကိုပဲ သုံးဖြစ်သွားတယ်။

- **Vite** ကို မြန်ဆန်တဲ့ Development Experience အတွက် သုံးတယ်။
- **React Router** နဲ့ `/` (Scanner အတွက်) နဲ့ `/admin` (QR Card ထုတ်ဖို့) ဆိုပြီး Route နှစ်ခုခွဲတယ်။
- **Tailwind CSS** ကိုသုံးပြီး **Dark "Hacker Terminal" Theme** လေး ရေးထားတယ်။ ကျွန်တော်က CS Mentor တစ်ယောက်ဆိုတော့ ဒီလို UI မျိုးက ပိုမိုက်တယ်လေ။ UI ကို Terminal တစ်ခုလို `root@404SNF:~# RCsys`, `[ EXECUTE_SAVE ]`, `SYSTEM_LOGS`, `>_` စသဖြင့် ဖန်တီးထားတော့ Developer တစ်ယောက်က သူ့အတွက်သူ ရေးထားတဲ့ ဆော့ဖ်ဝဲလ်လေးတစ်ခုလို ခံစားရစေတယ်။

အဓိက Magic ဖြစ်စေတဲ့ Library (၂) ခုကတော့ -

- **[qrcode.react](https://github.com/zpao/qrcode.react)** — Admin Page မှာ ကျောင်းသားတစ်ယောက်ချင်းစီအတွက် QR Canvas တွေ ထုတ်ပေးတယ်။ Card တိုင်းမှာ `CS-001` လို Student ID ကို ထည့်သွင်းထားပြီး Print ထုတ်ရလွယ်အောင် အဖြူရောင် Background ပေါ်မှာ နာမည်နဲ့ ID ကိုပါ တစ်ခါတည်း ရေးပေးထားတယ်။
- **[@yudiel/react-qr-scanner](https://github.com/yudielcurbelo/react-qr-scanner)** — Webcam ကိုဖွင့်ပြီး QR Code တွေကို Real-time ဖတ်ပေးတယ်။ သူကပဲ ကျွန်တော့် ကင်မရာကို Roll-call Machine တစ်ခုအဖြစ် ပြောင်းလဲပေးလိုက်တာပါ။

Card တွေကို Download လုပ်ဖို့အတွက် **JSZip + FileSaver** ကို သုံးထားလို့ Click တစ်ချက်နှိပ်လိုက်တာနဲ့ ကျောင်းသားအားလုံးရဲ့ Card တွေကို Print ထုတ်ဖို့အသင့် ZIP ဖိုင်အနေနဲ့ ရလာမှာပါ။

#### Backend — Express 5 (Database မလိုပါ)

Backend ကိုတော့ တမင်သက်သက် ရိုးရိုးရှင်းရှင်းလေးပဲ ထားထားပါတယ်။ `~/RCsys/` အောက်က Text ဖိုင်တွေကို ဖတ်/ရေး လုပ်ပေးမယ့် **Express** Server တစ်ခုပါပဲ။

```text
~/RCsys/
├── student/list.txt            # "CS-001, Aung Aung" — တစ်ကြောင်းလျှင် ကျောင်းသားတစ်ယောက်
└── daily/JS_Section_A_2026-07-05.txt   # နေ့စဉ် Attendance မှတ်တမ်းများ

```

Route နှစ်ခုပဲ ရှိပါတယ် -

* `GET /api/students` — ကျောင်းသားစာရင်းကို ယူဖို့။
* `POST /api/attendance` — ဖတ်လိုက်တဲ့ ID တွေနဲ့ အတန်းနာမည်ကို လက်ခံပြီး ဘယ်သူရှိတယ်/ပျက်တယ် တွက်ချက်ကာ ဖတ်လို့လွယ်တဲ့ Report တစ်ခု ထုတ်ပေးဖို့။

MongoDB တွေ၊ Postgres တွေ၊ ORM တွေ ဘာမှမလိုပါဘူး။ ကိုယ့်စက်ထဲမှာပဲ Run မယ့် စာသင်ခန်းသုံး Tool လေးတစ်ခုအတွက်ဆိုရင် Text ဖိုင်လေးတစ်ခုက ပိုစိတ်ချရတယ်၊ Backup လုပ်ရလွယ်တယ်၊ ပြီးတော့ Database အကြီးကြီးတွေအကြောင်းထက် ကျောင်းသားတွေကို ရှင်းပြရတာ ပိုလွယ်ကူပါတယ်။ တစ်ခါတလေမှာ အရိုးရှင်းဆုံး Tool က အမှန်ကန်ဆုံးပါပဲ။

Server က ပြုစုပြီးသား React App ကိုပါ တစ်ခါတည်း Serve လုပ်ပေးလို့ စနစ်တစ်ခုလုံးက **http://localhost:5000** ဆိုတဲ့ Process တစ်ခုတည်းမှာပဲ အလုပ်လုပ်ပါတယ်။

---

### အဆင့်ဆင့် တည်ဆောက်ခြင်း (Building It — Step by Step)

#### ၁။ ပထမဆုံး ကျောင်းသားစာရင်း (Student list)

Data တွေကို `~/RCsys/student/list.txt` ဆိုတဲ့ Text ဖိုင်လေးကနေ စပါတယ်။ ပထမဆုံး Run တဲ့အချိန်မှာ Server က Folder တွေကို အလိုအလျောက် ဆောက်ပေးပြီး စမ်းသပ်လို့ရအောင် Dummy စာရင်းတစ်ခုကိုပါ ဖန်တီးပေးပါတယ်။

#### ၂။ Admin Page (QR Generator)

API ကနေ စာရင်းကိုလှမ်းခေါ်၊ ကျောင်းသားတစ်ယောက်ချင်းစီအတွက် Map လုပ်ပြီး QR Canvas တွေကို အောက်ပါအတိုင်း Render လုပ်ပါတယ် -

```jsx
<QRCodeCanvas id="{`qr-${student.id}`}" size="{130}" value="{student.id}"/>

```

Card တွေကို နာမည်၊ ID တို့နဲ့ပေါင်းပြီး PNG အနေနဲ့ဖြစ်ဖြစ်၊ ZIP ဖိုင် တစ်ခုတည်းအနေနဲ့ဖြစ်ဖြစ် Export ထုတ်လို့ ရပါတယ်။

#### ၃။ Scanner Page (အပျော်ဆုံးအပိုင်း)

ဒါကတော့ App ရဲ့ နှလုံးသားပါပဲ။ `Scanner` Component က Code တွေကို အဆက်မပြတ် ဖတ်နေပေးပါတယ်။

* ဖတ်လိုက်တဲ့ Data ကို Trim လုပ်တယ်။
* `Set` တစ်ခုကို သုံးပြီး စစ်ဆေးတဲ့အတွက် ကျောင်းသားတစ်ယောက်တည်းက နှစ်ခါ ပြန်ဖတ်မိရင်တောင် မထပ်တော့ဘူး။
* ချက်ချင်း **Beep** အသံလေး မြည်သွားအောင် လုပ်ထားလို့ ကျွန်တော်ရော ကျောင်းသားပါ ဖတ်ပြီးသွားပြီဆိုတာကို ချက်ချင်း သိနိုင်တယ်။
* ဖတ်ပြီးတဲ့ ID ကို မျက်နှာပြင်ပေါ်က `SYSTEM_LOGS` ထဲကို ထည့်ပေးတယ်။

```jsx
if (!scannedSet.current.has(studentId)) {
  scannedSet.current.add(studentId);
  playBeep();
  setScannedIds(Array.from(scannedSet.current));
}

```

#### ၄။ Roll call ကို သိမ်းဆည်းခြင်း

Scan ဖတ်လို့ ပြီးသွားရင် `[ EXECUTE_SAVE ]` ကို နှိပ်လိုက်ပါတယ်။ Session နာမည် (ဥပမာ `JS_Section_A`) ကို တောင်းမယ့် Modal လေးကျလာပြီး Frontend က ရရှိလာတဲ့ ID တွေကို POST လုပ်ပေးပါတယ်။ Server က -

* စာရင်းထဲမှာ ပါပေမယ့် Scan မဖတ်ထားတဲ့သူတွေကို **Absent (ပျက်)** အဖြစ် မှတ်လိုက်တယ်။
* `~/RCsys/daily/` ထဲမှာ နေ့စွဲနဲ့ Report ဖိုင် တစ်ခုကို သိမ်းပေးတယ်။
* ပြီးရင် ရလဒ်ကို ပြန်ပို့ပေးတယ်။

အဲဒီအခါ UI မှာ ဘယ်သူတွေ ပျက်လဲဆိုတဲ့ **Missing List** ကို အတိအကျ ပြပေးတဲ့အတွက် ကျောင်းသားတွေ အတန်းထဲက မထွက်ခင်မှာပဲ ဘယ်သူတွေ မလာဘူးလဲဆိုတာ ကျွန်တော် ချက်ချင်း သိလိုက်ရပါပြီ။

---

### ကြုံတွေ့ခဲ့ရသော အခက်အခဲများ (Challenges I Hit Along the Way)

1. **Duplicate scans** — Card တစ်ခုတည်းကို နှစ်ခါဖတ်မိတာမျိုးက သေချာပေါက် ကြုံရပါတယ်။ ဒါကို `Set` နဲ့ Beep အသံလေး သုံးပြီး ဖြေရှင်းခဲ့တယ်။
2. **Report filenames** — Session နာမည်တွေမှာ Space တွေနဲ့ မြန်မာစာတွေ ပါလာတတ်ပါတယ်။ ဒါကြောင့် ဖိုင်နာမည်ကို `[A-Za-z0-9_\u1000-\u109F]` လို့ Sanitize လုပ်ပြီး မြန်မာစာလည်း ရသလို၊ ဖိုင်နာမည်တွေလည်း ပြဿနာမတက်အောင် လုပ်ခဲ့တယ်။
3. **Client-side routing on the server** — React Router ကြောင့် `/admin` ကို Refresh လုပ်တဲ့အခါ 404 Error မတက်အောင် Server ဘက်ကနေ API မဟုတ်တဲ့ Route တွေအားလုံးကို `index.html` ဆီ ပြန်လွှဲပေးရတယ်။
```js
app.get(/.*/, (req, res) => res.sendFile(path.join(distPath, "index.html")));

```


4. **CORS ပြဿနာ** — Dev လုပ်နေတုန်း Vite Dev Server (:5173) နဲ့ Express API (:5000) ကြား ချိတ်ဆက်ဖို့ Backend ကနေ `cors()` ကို Enable လုပ်ပေးခဲ့ရတယ်။
5. **Beep အသံ** — အသံဖိုင် သက်သက်ထည့်မယ့်အစား Web Audio API ကိုသုံးပြီး Sine beep လေး ဖန်တီးခဲ့တယ်။ သေးငယ်တဲ့ အပြောင်းအလဲလေးပေမယ့် Scan ဖတ်တဲ့အခါ တကယ်ကို အသက်ဝင်သွားစေတယ်။

---

### ရရှိလာတဲ့ ရလဒ် (The Result)

နာမည်ခေါ်ဖို့ **၅ မိနစ်ကျော် အော်နေရတဲ့ အလုပ်ကနေ စက္ကန့် ၃၀ လောက် Scan ဖတ်ရုံနဲ့** ပြီးသွားပါပြီ။

* ကျောင်းသားတွေကလည်း လေယာဉ်ပေါ်တက်သလို Card လေးတွေနဲ့ Scan ဖတ်ရတာကို သဘောကျကြတယ်။
* Manual တွက်စရာမလိုဘဲ တိကျတဲ့ Present/Absent စာရင်းကို ရလာတယ်။
* Report တွေက Text ဖိုင်တွေအနေနဲ့ သိမ်းထားလို့ မှတ်တမ်းတွေထဲ တိုက်ရိုက် ကူးထည့်ရုံပါပဲ။
* Database မလို၊ အင်တာနက် မလို၊ Deploy လုပ်စရာ မလိုဘဲ ကျွန်တော့် Laptop ပေါ်မှာတင် အလုပ်လုပ်ပါတယ်။

---

### ကျွန်တော် ဘာတွေသင်ယူခဲ့ရလဲ (What I Learned)

* **ကိုယ့်အခက်အခဲအတွက် ကိုယ်တိုင် Tool ဖန်တီးပါ။** အတန်းပြီးတိုင်း ကြုံနေရတဲ့ ပြဿနာက ဒီ Project အတွက် အကောင်းဆုံး Spec တစ်ခု ဖြစ်လာခဲ့တယ်။
* **QR code ဆိုတာ String တစ်ခုပါပဲ။** String ID တစ်ခုကို QR အဖြစ် ပြောင်းလို့ရမှန်း သိသွားတဲ့အခါ စနစ်တစ်ခုလုံးက *String → ကင်မရာ → တိုက်စစ်မယ် → သိမ်းမယ်* ဆိုတဲ့ ရိုးရှင်းတဲ့ လုပ်ငန်းစဉ်လေး ဖြစ်သွားတယ်။
* **Simple storage က အကောင်းဆုံးပါပဲ။** Database သုံးဖို့ အရင်ဆုံး စဉ်းစားမိပေမယ့် တကယ်တမ်း အဖြေမှန်က Text ဖိုင်လေးပါပဲ။
* **Good UX ဆိုတာ သေးငယ်တဲ့ အသေးစိတ်လေးတွေပါ။** Beep မြည်တာ၊ ထပ်နေတာတွေကို ဖယ်တာ၊ Terminal ပုံစံလေး ဖန်တီးတာ၊ ZIP နဲ့ ဒေါင်းလုဒ်ဆွဲတာတွေက မပါမဖြစ် မဟုတ်ပေမယ့် ဒါတွေရှိနေလို့ Tool လေးက သုံးရတာ အရမ်းမိုက်သွားတာပါ။
* **လက်တွေ့ ဥပမာပြပါ။** ကျောင်းသားတွေကို သင်ပေးနေတဲ့ Stack (React + Express) ကိုပဲ တိုက်ရိုက် ပြန်သုံးပြလိုက်လို့ ကျောင်းသားတွေအတွက် ကျောက်သင်ပုန်းပေါ်မှာ သင်ခဲ့ရတဲ့ CRUD သဘောတရားတွေကို လက်တွေ့ အလုပ်လုပ်နေတဲ့ ဥပမာတစ်ခုအဖြစ် တွေ့မြင်သွားရပါတယ်။

---

### စမ်းသပ်ကြည့်ဖို့ (Try It)

ဒီ Project က Open-source ပါ။ Code တွေကို ယူပါ၊ မင်းရဲ့ ကိုယ်ပိုင် `~/RCsys/student/list.txt` လေးကို ထည့်ပြီး နောက်အတန်းချိန်တွေမှာ စက္ကန့်ပိုင်းအတွင်း Roll call လုပ်လိုက်ပါ။

* **Author:** Aster Julian Ray — Senior Mentor, UCS-Pyay
* **GitHub:** [@picakhant](https://github.com/picakhant)

*Roll call ခေါ်ဖို့ အော်နေစရာ မလိုတော့ပါဘူး။ လုံးဝပါပဲ။*

- **Author:** Aster Julian Ray — Senior Mentor, UCS-Pyay
- **GitHub:** [@picakhant](https://github.com/picakhant)
- **Portfolio:** [Aster Julian Ray](https://aster-dev.vercel.app)
