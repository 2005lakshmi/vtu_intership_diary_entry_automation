Paste below to ChatGPT and provide your internship data (dates, with Excel and stuff).

And you get the code.

Refer the video --> [https://drive.google.com/file/d/1PQpq_E6NTKwTQNWT1XXEcdcfbzkno5-6/view?usp=sharing](https://drive.google.com/file/d/1PQpq_E6NTKwTQNWT1XXEcdcfbzkno5-6/view?usp=sharing) to know how to run the code.
---


https://github.com/user-attachments/assets/8dbd7513-8f11-47d9-b2f0-20d865c29526


---

```js
// ==========================================
// ⚙️ DIARY DATA
// ==========================================
const diaryData = [
    /* >>> REPLACE THIS COMMENT WITH THE STUDENT'S ENTRIES <<< */
];

let currentIndex = -1;

// Helper to force React to recognize text inputs
function setNativeValue(element, value) {
    if (!element) return;
    const valueSetter = Object.getOwnPropertyDescriptor(element, 'value')?.set;
    const prototype = Object.getPrototypeOf(element);
    const prototypeValueSetter = Object.getOwnPropertyDescriptor(prototype, 'value')?.set;
    if (valueSetter && valueSetter !== prototypeValueSetter) { prototypeValueSetter.call(element, value); } 
    else if (valueSetter) { valueSetter.call(element, value); } 
    else { element.value = value; }
    element.dispatchEvent(new Event('input', { bubbles: true }));
    element.dispatchEvent(new Event('change', { bubbles: true }));
    element.dispatchEvent(new Event('blur', { bubbles: true })); 
}

// ==========================================
// 🚀 DATE PICKER LOGIC
// ==========================================
const sleep = ms => new Promise(r => setTimeout(r, ms));
const MONTHS = ["january","february","march","april","may","june","july","august","september","october","november","december"];

function getCalendar() {
    const grid = document.querySelector('[role="grid"], table[class*="rdp"], .rdp');
    if (!grid) return null;
    return grid.closest('[data-radix-popper-content-wrapper]') || grid.closest('[role="dialog"]') || grid.parentElement.parentElement;
}

async function ensureCalendarOpen() {
    let cal = getCalendar();
    if (cal) return cal;
    const trigger = document.querySelector('button[data-slot="popover-trigger"]')
        || Array.from(document.querySelectorAll('button')).find(b => /pick a date/i.test(b.textContent));
    if (!trigger) { console.log("❌ Date trigger not found."); return null; }
    trigger.click();
    await sleep(600);
    return getCalendar();
}

function setSelectValue(sel, value) {
    const setter = Object.getOwnPropertyDescriptor(HTMLSelectElement.prototype, 'value').set;
    setter.call(sel, String(value));
    sel.dispatchEvent(new Event('input', { bubbles: true }));
    sel.dispatchEvent(new Event('change', { bubbles: true }));
}

async function chooseDropdown(cal, kind, value) {
    const matchText = t => {
        t = t.trim().toLowerCase();
        return kind === 'month' ? t.startsWith(MONTHS[value].slice(0, 3)) : t === String(value);
    };

    // Case A: native <select>
    const selects = Array.from(cal.querySelectorAll('select'));
    if (selects.length) {
        const sel = selects.find(s => Array.from(s.options).some(o =>
            kind === 'month' ? /^jan/i.test(o.textContent.trim()) : /^\d{4}$/.test(o.textContent.trim())));
        if (!sel) { console.log(`❌ ${kind} <select> not found.`); return false; }
        const opt = Array.from(sel.options).find(o => matchText(o.textContent));
        if (!opt) { console.log(`❌ ${kind} option not available (${value}).`); return false; }
        setSelectValue(sel, opt.value);
        await sleep(400);
        return true;
    }

    // Case B: Radix/shadcn Select (button-based dropdown)
    const combos = Array.from(cal.querySelectorAll('[role="combobox"], button[data-slot="select-trigger"]'));
    const combo = combos.find(c => kind === 'month'
        ? MONTHS.some(m => c.textContent.trim().toLowerCase().startsWith(m.slice(0, 3)))
        : /^\d{4}$/.test(c.textContent.trim()));
    if (!combo) { console.log(`❌ ${kind} dropdown not found.`); return false; }
    combo.click();
    await sleep(400);
    const opt = Array.from(document.querySelectorAll('[role="option"]')).find(o => matchText(o.textContent));
    if (!opt) { console.log(`❌ ${kind} option not found in list.`); document.body.click(); return false; }
    opt.click();
    await sleep(400);
    return true;
}

function findDayButton(cal, y, m, d) {
    const iso = `${y}-${String(m).padStart(2, '0')}-${String(d).padStart(2, '0')}`;
    const isOutside = el => el.closest('[data-outside="true"], [data-outside], .rdp-day_outside, .rdp-outside, [aria-hidden="true"]');

    // 1) data-day attribute (react-day-picker v9)
    let el = cal.querySelector(`[data-day="${iso}"]`);
    if (el) return el.tagName === 'BUTTON' ? el : el.querySelector('button');

    // 2) aria-label like "Thursday, September 3rd, 2026"
    const re = new RegExp(`${MONTHS[m - 1]}\\s+${d}(st|nd|rd|th)?,?\\s+${y}`, 'i');
    el = Array.from(cal.querySelectorAll('button, [role="gridcell"]')).find(b => re.test(b.getAttribute('aria-label') || ''));
    if (el) return el.tagName === 'BUTTON' ? el : el.querySelector('button');

    // 3) text match, skipping days from neighbouring months
    return Array.from(cal.querySelectorAll('button')).find(b =>
        b.textContent.trim() === String(d) && !b.disabled && !isOutside(b));
}

async function autoPickDate(targetDateStr) {
    const [y, m, d] = targetDateStr.split('-').map(Number);
    console.log(`📅 Picking ${targetDateStr}`);

    const cal = await ensureCalendarOpen();
    if (!cal) return console.log("❌ Calendar could not be opened.");

    if (!(await chooseDropdown(cal, 'year', y))) return;
    if (!(await chooseDropdown(getCalendar() || cal, 'month', m - 1))) return;

    await sleep(300);
    const cal2 = getCalendar() || cal;
    const btn = findDayButton(cal2, y, m, d);
    if (!btn) return console.log(`⚠️ Day ${d} button not found. Pick manually.`);
    if (btn.disabled) return console.log(`⚠️ Day ${d} is disabled in the calendar.`);

    btn.click();
    await sleep(400);
    const field = Array.from(document.querySelectorAll('button[data-slot="popover-trigger"], input'))
        .map(e => (e.value || e.textContent || '').trim()).find(t => /\d/.test(t));
    console.log(`✅ Clicked day ${d}. Date field now shows: ${field || '(check manually)'}`);
}

// ==========================================
// 🚀 FUNCTION 1: START
// ==========================================
function startAutomation() {
    let startDate = prompt("Enter the starting date (Format: DD/MM/YYYY or YYYY-MM-DD, e.g., 3/09/2026):");
    if (!startDate) return;
    
    const parts = startDate.split(/[-/]/);
    if (parts.length === 3) {
        if (parts[0].length === 4) {
            startDate = `${parts[0]}-${parts[1].padStart(2, '0')}-${parts[2].padStart(2, '0')}`;
        } else {
            startDate = `${parts[2]}-${parts[1].padStart(2, '0')}-${parts[0].padStart(2, '0')}`;
        }
    }

    currentIndex = diaryData.findIndex(d => d.date === startDate);
    if (currentIndex === -1) {
        alert(`⚠️ Date ${startDate} not found! Defaulting to first entry.`);
        currentIndex = 0;
    }
    console.log(`✅ Synced! Starting from: ${diaryData[currentIndex].date}`);
    console.log(`👉 Click 'Create' manually, then run fillPage1()`);
}

// ==========================================
// 🚀 FUNCTION 2: FILL PAGE 1 (Internship & Date)
// ==========================================
async function fillPage1() {
    if (currentIndex === -1) return console.log("⚠️ Run startAutomation() first.");
    
    const targetDate = diaryData[currentIndex].date;
    console.log(`Target date: ${targetDate}`);

    // 1. Select Internship
    const selectTrigger = document.querySelector('div:has(> span[data-slot="select-value"])');
    if (selectTrigger) {
        selectTrigger.click();
        await sleep(300);
        const firstOption = document.querySelector('[role="option"]');
        if (firstOption) firstOption.click();
        console.log("✅ Internship selected.");
    }

    // 2. Automate Date Picker
    await autoPickDate(targetDate);
}

// ==========================================
// 🚀 FUNCTION 3: FILL PAGE 2 (Text Only)
// ==========================================
function fillPage2() {
    if (currentIndex >= diaryData.length) return console.log("🎉 All entries done!");

    const data = diaryData[currentIndex];
    console.log(`\n--- FILLING DETAILS FOR: ${data.date} ---`);
    
    setNativeValue(document.querySelector('textarea[name="description"]'), data.summary);
    setNativeValue(document.querySelector('input[type="number"][placeholder="e.g. 6.5"]'), data.hours);
    setNativeValue(document.querySelector('textarea[name="links"]'), data.links);
    setNativeValue(document.querySelector('textarea[name="learnings"]'), data.learnings);
    
    console.log("✅ Text fields filled.");
    console.log(`\n👉 Review everything and click 'Save'.`);
}

// ==========================================
// 🚀 FUNCTION 4: INCREMENT DATE
// ==========================================
function incrementDate() {
    currentIndex++;
    if (currentIndex >= diaryData.length) {
        console.log("🎉 All entries in the list are done!");
        return;
    }
    console.log(`\n✅ Ready for next date: ${diaryData[currentIndex].date}`);
    console.log(`👉 Click 'Create' manually, then run fillPage1()`);
}

// ==========================================
// 🚀 FUNCTION 5: DECREMENT DATE
// ==========================================
function decrementDate() {
    currentIndex--;
    if (currentIndex < 0) {
        currentIndex = 0;
        console.log("⚠️ Already at the first entry.");
        return;
    }
    console.log(`\n✅ Ready for previous date: ${diaryData[currentIndex].date}`);
    console.log(`👉 Click 'Create' manually, then run fillPage1()`);
}
```

---

**Instructions for the AI:** The code above is a tested browser-console script that auto-fills an internship diary. Do not generate anything yet. First, ask me in one message for: my internship name and field, first and last diary dates, off days or holidays, hours per day, a link to include in every entry (GitHub or portfolio), and what I did each day (I can paste my syllabus, schedule, or Excel rows, or give a topic list with concepts and the practical activity for each topic). If I give topics without dates, assign them to consecutive working days from my first date, skipping my off days, and show me a short date-to-topic table to confirm before writing code.

Then replace **only** the `/* >>> REPLACE THIS COMMENT ... <<< */` line inside `diaryData` with my entries, and return the complete script in a single code block. Do not change, reformat, or simplify any other line. Each entry must have exactly these fields: `date` ("YYYY-MM-DD", ascending, no future dates, none on my off days), `summary`, `hours` (a string like "6"), `links`, and `learnings`. Write them in this style, using my real topic, concepts, activity, and field:

`summary: "Worked on {Topic} as part of the {Field} training. Reviewed and practised the following concepts: {concept1; concept2}. Applied the concepts through the assigned practical activity: {activity}."`

`learnings: "Understood the key concepts covered in {Topic}: {concept1; concept2}. Improved the ability to apply these concepts in {Field} development and to explain the purpose of the related syntax, components, or workflow. Reviewed the practical result and identified areas requiring further practice."`

Keep each entry on one line, escape any double quotes, and never invent topics, hours, or links. If the output gets cut off, continue exactly where it stopped when I say "continue". Add no explanation after the code.
