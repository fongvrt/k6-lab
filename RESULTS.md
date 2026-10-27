# K6 lab results

## Part 0
- Step 0.1, http_req_duration: 328.63 ms (มีแค่ 1 request ค่า avg = min = med = max = p(90) = p(95) = 328.63 ms เท่ากันหมด) checks_succeeded 100.00% (1 out of 1), http_reqs 1, iterations 1
- Step 0.2, requests, rate and p(95), and why the rate stays below 5: ได้ 75 requests, rate 3.67 requests/s, p(95) = 319.39 ms (failed 0%, checks ผ่าน 75/75) สาเหตุที่ rate ต่ำกว่า 5 คือ VU หนึ่งตัวไม่ได้ส่ง 1 request ต่อวินาทีพอดี ใน 1 iteration มีทั้งตัว request (เฉลี่ย 299.66 ms) และ `sleep(1)` อีก 1 วินาที ทำให้ iteration_duration เฉลี่ย 1.34 s ดังนั้น VU หนึ่งตัวส่งได้ประมาณ 1 / 1.34 ≈ 0.75 requests/s และ 5 VUs ส่งได้ประมาณ 5 × 0.75 ≈ 3.7 requests/s ซึ่งตรงกับค่าที่วัดได้ 3.67/s ถ้าจะได้ 5/s เต็ม server ต้องตอบได้โดยไม่ใช้เวลาเลย
- Step 0.6, exit code of the normal run and of the run with P95=300: รันปกติได้ exit code **0** (ผ่านทั้ง 3 thresholds: checks 100.00%, p(95)=463.1ms < 1500, http_req_failed 0.00%) รันด้วย `-e P95=300` ได้ exit code **99** (`✗ 'p(95)<300' p(95)=461.54ms` และ K6 แสดงข้อความ "thresholds on metrics 'http_req_duration' have been crossed" ส่วนอีก 2 thresholds ยังผ่าน)
- Step 0.7, rules that passed, p(95), p(99), exit code after your change: `07-requirement.js` ตั้ง `vus: 3`, `duration: '30s'`, `http_req_duration: ['p(95)<1000', 'p(99)<1500']` และ `http_req_failed: ['rate<0.01']` (เพิ่ม `summaryTrendStats` เพื่อให้ผลสรุปแสดง p(99)) ผ่านทั้ง 3 rules: p(95) = 495.43 ms < 1000, p(99) = 584.87 ms < 1500, http_req_failed = 0.00% < 1% (65 requests) exit code 0 จากนั้นเปลี่ยนตัวเลขหนึ่งตัวใน requirement จาก "95% ภายใน 1 วินาที" เป็น "95% ภายใน 0.4 วินาที" (`p(95)<400`) แล้วรันใหม่ ได้ `✗ 'p(95)<400' p(95)=482.15ms` อีก 2 rules ยังผ่าน และ exit code เป็น **99** หลังจากนั้นแก้สคริปต์กลับเป็น `p(95)<1000` ตามเดิม
- Step 0.8, what was wrong, and how the checks showed it: **ก่อนแก้** รัน 5 VUs 20 วินาที ได้ 70 requests ทุกตัวได้ status 200 และ http_req_failed 0% แต่ checks_succeeded เหลือแค่ 81.42% (171/210) โดย `no pepperoni` ผ่านแค่ 88% (✓ 62 / ✗ 8) และ `no knife needed` ผ่านแค่ 55% (✓ 39 / ✗ 31) **สิ่งที่ผิด** คือสคริปต์ส่งชื่อเป็นตัวพิมพ์เล็ก `excludedIngredients: ['pepperoni']` และ `excludedTools: ['knife']` แต่ `explore.js` แสดงว่า API ใช้ชื่อที่ขึ้นต้นด้วยตัวพิมพ์ใหญ่ คือ `"Pepperoni"` (topping ID 9) และ `"Knife"` (tools คือ `["Knife","Pizza cutter","Scissors"]`) API เทียบชื่อแบบตรงตัวอักษร (case-sensitive) จึงไม่รู้จัก `pepperoni` และ `knife` และไม่ได้กรองอะไรออกเลย แต่ก็ไม่แจ้ง error ยังตอบ 200 เหมือนปกติ **แก้** โดยเปลี่ยนเป็น `['Pepperoni']` และ `['Knife']` แล้วรันใหม่แบบเดิม ผ่าน 100% (225/225 checks, 75 requests) **ทำไม checks ตอนรันโหลดถึงเจอ** เพราะ QuickPizza สุ่มพิซซ่าใหม่ทุกครั้ง พิซซ่าส่วนใหญ่บังเอิญไม่มี pepperoni (88%) และประมาณครึ่งหนึ่งบังเอิญใช้เครื่องมืออื่นที่ไม่ใช่มีด ถ้าลองส่ง request ด้วยมือแค่ครั้งเดียว มีโอกาสสูงที่จะได้พิซซ่าที่ดูถูกต้องและสรุปผิดว่า API ทำงานถูก แต่การรัน 70 requests พร้อม check ที่ดูเนื้อหาของคำตอบทุกครั้ง ทำให้เห็นความผิดพลาดเป็นเปอร์เซ็นต์ชัดเจน และ status 200 อย่างเดียวไม่พอ เพราะ API ตอบเร็วและไม่ error แต่ให้ผลลัพธ์ผิด

## Part A results
| Step | Test type    | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1   | Smoke        | 1     | 0.49  | 583.97 ms | 667.3 ms | 0.00% (0/15) | 100.00% (20/20) |
| A2   | Average load | 3     |       |       |     |        |        |
| A3   | Stress       | 5     |       |       |     |        |        |
| A4   | Spike        | 5     |       |       |     |        |        |
| A5   | Soak         | 2     |       |       |     |        |        |

## Part B
1.
2.
3.
4.
5.
6.
7.
8.
9.
