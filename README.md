# F.CSA313 — Лаборатори №3
## Чанарын сценарио → SLO → k6 Threshold

### Оюутны мэдээлэл

- **Нэр:** Э. Мөнх-Очир
- **Оюутны код:** B232270022
- **Хичээл:** F.CSA313 — Software Quality Assurance and Testing
- **Лаборатори:** Лаборатори №3
- **Сэдэв:** Чанарын сценарио → SLO → k6 threshold
- **Платформ:** macOS
- **Testing tool:** Grafana k6 v2.2.0
- **Runtime:** Node.js + Express

---

## 1. Лабораторийн зорилго

Энэхүү лабораторийн ажлын зорилго нь системийн чанарын шаардлагыг чанарын сценари хэлбэрээр тодорхойлж, тэдгээрийг Service Level Objective (SLO) болгон хөрвүүлэн, Grafana k6 ашиглан performance, reliability болон availability-ийн хэмжилт хийхэд оршино.

Лабораторийн хүрээнд Express.js дээр энгийн REST API үүсгэж, уг API-ийн endpoint-үүд дээр k6 ашиглан threshold-based performance testing хийсэн.

Мөн дараах туршилтуудыг гүйцэтгэсэн:

1. SLO threshold PASS test
2. Chaos / availability test
3. Deliberate threshold FAIL test

---

## 2. Системийн бүтэц

Энэ лабораторид дараах гурван үндсэн API endpoint ашигласан.

| Method | Endpoint      | Үүрэг                                              |
|--------|---------------|-----------------------------------------------------|
| POST   | `/cart/add`   | Сагсанд бараа нэмэх                                 |
| GET    | `/report`     | Тайлан авах, 200–400 ms random delay-тэй            |
| POST   | `/pay`        | Төлбөр хийх, ойролцоогоор 5% HTTP 500 алдаа гаргана |

Систем локал орчинд дараах хаягаар ажилласан:

```
http://localhost:3000
```

---

## 3. API Endpoint-үүд

### 3.1 POST `/cart/add`

Энэ endpoint нь хурдан ажиллах зориулалттай.

Жишээ response:

```json
{
  "ok": true,
  "items": 1
}
```

Энэ endpoint-ийн performance requirement:

```
p(95) < 200 ms
```

### 3.2 GET `/report`

Энэ endpoint нь тайлангийн мэдээлэл буцаах бөгөөд 200–400 ms-ийн хооронд санамсаргүй delay үүсгэдэг.

Жишээ response:

```json
{
  "rows": 20000
}
```

Үндсэн SLO:

```
p(95) < 450 ms
```

### 3.3 POST `/pay`

Энэ endpoint нь төлбөрийн үйлдлийг дуурайлган ажиллана.

Ойролцоогоор 5% request HTTP 500 алдаа буцаана.

Амжилттай үед:

```json
{
  "paid": true
}
```

Алдаатай үед:

```json
{
  "error": "gateway timeout"
}
```

Reliability SLO:

```
Error rate < 8%
```

---

## 4. Quality Scenarios

### 4.1 Performance Scenario — `/cart/add`

1. **Overview** — Хэрэглэгч сагсанд бараа нэмэх үед систем хурдан response өгөх шаардлагатай.
2. **System State** — Express сервер ажиллаж байгаа бөгөөд `/cart/add` endpoint ашиглах боломжтой байна.
3. **Environment State** — Систем локал macOS орчинд, `localhost:3000` дээр ажиллана.
4. **External Stimulus** — 20 virtual user `/cart/add` endpoint рүү зэрэг request илгээнэ.
5. **Required Response** — Систем HTTP 200 status code буцааж, request-ийг хурдан боловсруулах ёстой.
6. **Measure** — 95-р percentile response time:

```
p(95) < 200 ms
```

### 4.2 Reliability Scenario — `/pay`

1. **Overview** — Төлбөрийн endpoint нь request боловсруулах үед алдаа гаргах боломжтой тул reliability-г хэмжинэ.
2. **System State** — Express сервер ажиллаж байгаа бөгөөд `/pay` endpoint идэвхтэй байна.
3. **Environment State** — 20 virtual user систем рүү зэрэг request илгээнэ.
4. **External Stimulus** — k6 олон удаагийн POST request-ийг `/pay` endpoint рүү илгээнэ.
5. **Required Response** — Амжилттай request HTTP 200 status code буцаах ёстой. HTTP 500 алдааны хэмжээ SLO-оос хэтрэхгүй байна.
6. **Measure** — HTTP error rate:

```
error rate < 8%
```

### 4.3 Availability Scenario — Server Crash / Restart

1. **Overview** — Сервер түр хугацаанд unavailable болсон үед системийн request-based availability-г хэмжинэ.
2. **System State** — Express сервер ажиллаж байгаа бөгөөд k6 20 virtual user ашиглан request илгээнэ.
3. **Environment State** — Туршилтын явцад серверийг зориудаар 10 секундын хугацаанд зогсоож, дараа нь дахин ажиллуулсан.
4. **External Stimulus** — Server crash / shutdown үүсгэж, request-үүдийг үргэлжлүүлэн илгээнэ.
5. **Required Response** — Сервер дахин ассны дараа request-үүд дахин амжилттай боловсруулагдах ёстой.
6. **Measure** — Availability:

```
Availability = Successful Requests / Total Requests × 100%
```

SLO:

```
Availability >= 90%
```

---

## 5. SLO Definition

Энэ лабораторид дараах SLO-уудыг тодорхойлсон.

| Scenario / Endpoint | SLI                          | SLO Threshold | Window / Condition |
|----------------------|-------------------------------|----------------|----------------------|
| `/cart/add`          | Response time                 | p95 < 200 ms   | k6 load test         |
| `/pay`                | Error rate                    | < 8%           | k6 load test         |
| `/report`             | Response time                 | p95 < 450 ms   | k6 load test         |
| Overall checks        | Successful checks             | > 90%          | k6 load test         |
| Availability           | Successful / total requests   | >= 90%         | Chaos test           |

---

## 6. Threshold Rationale

### `/cart/add`

`/cart/add` нь хурдан endpoint байх ёстой тул:

```
p(95) < 200 ms
```

гэсэн threshold сонгосон.

### `/pay`

`/pay` endpoint нь серверийн кодоор ойролцоогоор 5% HTTP 500 алдаа гаргах боломжтой. Иймээс туршилтын SLO-г:

```
error rate < 8%
```

гэж тодорхойлсон.

### `/report`

`/report` endpoint нь 200–400 ms random delay-тэй. Иймээс 95-р percentile-ийн response time-д бага зэрэг нэмэлт margin өгөхийн тулд:

```
p(95) < 450 ms
```

гэсэн threshold сонгосон.

### Overall checks

Endpoint-үүдийн functional checks-ийн амжилтыг:

```
checks rate > 90%
```

гэж тодорхойлсон.

### Availability

Chaos test-ийн үед:

```
Availability >= 90%
```

гэсэн SLO ашигласан.

---

## 7. Lab 2-той холбоо

Энэхүү лаборатори нь өмнөх Lab 2-ийн performance testing ажлыг үргэлжлүүлсэн.

Lab 2 дээр тогтоосон performance SLO:

```
p(95) < 406.08 ms
Error rate < 1%
```

Lab 2-ийн baseline test-ийн actual p95:

```
282.41 ms
```

Мөн Lab 2-ийн threshold PASS test-ийн actual p95:

```
402.98 ms
```

Lab 3 дээр `/report` endpoint нь 200–400 ms random delay-тэй тусгай endpoint тул endpoint-ийн өөрийн behavior-д үндэслэн:

```
/report p(95) < 450 ms
```

гэсэн SLO ашигласан.

Ингэснээр Lab 2-ийн performance testing туршлагыг Lab 3-ийн endpoint-specific SLO болон k6 threshold testing-тэй холбосон.

---

## 8. k6 Threshold Test

Үндсэн threshold test файл:

```
slo-test.js
```

Үндсэн thresholds:

```js
thresholds: {
  'http_req_duration{name:cart}': ['p(95)<200'],
  'http_req_failed{name:pay}': ['rate<0.08'],
  'checks': ['rate>0.90'],
  'http_req_duration{name:report}': ['p(95)<450'],
}
```

Үндсэн PASS test:

- 20 VUs
- 1 minute

---

## 9. PASS Test

PASS test-ийн үед бүх үндсэн SLO threshold хангагдсан.

### 9.1 Results

**`/cart/add`**
- p(95) = 1.78 ms
- Threshold = p(95) < 200 ms
- Result = **PASS**

**`/report`**
- p(95) = 392.48 ms
- Threshold = p(95) < 450 ms
- Result = **PASS**

**`/pay`**
- Error rate = 5.50%
- Threshold = error rate < 8%
- Result = **PASS**

**Checks**
- Checks = 98.16%
- Threshold = rate > 90%
- Result = **PASS**

**Overall**
- Total HTTP requests = 2781
- Iterations = 927
- VUs = 20
- Duration = 1 minute

PASS test-ийн үед үндсэн SLO-ууд бүгд threshold дотор байсан.

---

## 10. PASS Screenshot

PASS test-ийн screenshot:

```
screenshots/pass.png
```

---

## 11. Chaos / Availability Test

Availability болон reliability-ийн behavior-ийг шалгахын тулд 2 минутын chaos test хийсэн.

Туршилтын явцад серверийг зориудаар түр зогсоож, дараа нь дахин ажиллуулсан.

Chaos test-ийн configuration:

- VUs = 20
- Duration = 2 minutes
- Server downtime = 10 seconds

---

## 12. Chaos Test Results

- Нийт request: 5733
- Амжилттай request: 4828
- Амжилтгүй request: 905

Availability:

```
Availability
= Successful Requests / Total Requests × 100
= 4828 / 5733 × 100
= 84.21%
```

SLO: `Availability >= 90%`

Actual: `84.21%`

Тиймээс chaos test-ийн availability SLO хангагдаагүй.

---

## 13. Chaos Test — Endpoint Results

**`/cart/add`**
- p(95) = 1.52 ms

**`/report`**
- p(95) = 390.97 ms

**`/pay`**
- Error rate = 18.36%
- Failed = 351 / 1911

**Checks**
- Checks = 84.21%
- Succeeded = 4828 / 5733
- Failed = 905 / 5733

**Overall**
- Total HTTP requests = 5733
- Iterations = 1911
- Maximum VUs = 20
- Duration = 2 minutes
- Overall p(95) = 365.34 ms

Chaos test-ийн үед `/cart/add` болон `/report` response time-ийн threshold хангагдсан боловч availability болон `/pay` reliability threshold хангагдаагүй.

---

## 14. Chaos Screenshot

Chaos test-ийн screenshot:

```
screenshots/chaos.png
```

---

## 15. Availability Error Budget

Availability SLO: `Availability >= 90%`

Иймээс зөвшөөрөгдөх failure rate: `100% - 90% = 10%`

Chaos test-ийн нийт request: `5733`

10%-ийн request-based error budget:

```
5733 × 0.10 = 573.3 requests
```

Ойролцоогоор: **573 failed requests** зөвшөөрөгдөнө.

Actual failed requests: `905`

Тиймээс request-based error budget-аас:

```
905 - 573 = 332 requests
```

орчим илүү failure гарсан.

Actual availability: `84.21%` байсан тул 90%-ийн availability SLO хангагдаагүй.

---

## 16. Time-Based болон Request-Based Error Budget

Availability error budget-ийг хоёр өөр байдлаар ойлгож болно.

### Time-Based Budget

2 минутын туршилтад 90% availability шаардлагатай бол:

```
2 minutes × 10%
= 0.2 minutes
= 12 seconds
```

Иймээс time-based байдлаар хамгийн ихдээ ойролцоогоор **12 seconds** unavailable байх боломжтой.

### Request-Based Budget

Энэ лабораторид request-based availability ашигласан.

- Нийт request: 5733
- Зөвшөөрөгдөх failure: `5733 × 10% = 573.3`
- Actual failure: 905

Иймээс request-based budget хэтэрсэн.

### Ялгаа

Time-based budget нь систем хэдэн секунд unavailable байсан гэдгийг хэмждэг.

Request-based budget нь нийт request-үүдээс хэд нь амжилтгүй болсон гэдгийг хэмждэг.

Энэ лабораторийн шаардлагын дагуу availability-г:

```
Successful Requests / Total Requests
```

томъёогоор тооцсон.

---

## 17. Availability ба Reliability-ийн харилцан хамаарал

Availability болон reliability нь хоорондоо холбоотой боловч ижил хэмжүүр биш.

Availability нь систем request хүлээн авч, үйлчилгээ үзүүлж чадсан эсэхийг хэмждэг.

Reliability нь request боловсруулах явцад алдаа гарсан эсэхийг хэмждэг.

Chaos test-ийн үед:

- Availability = 84.21%
- `/pay` endpoint-ийн error rate = 18.36%

Энэ нь серверийн түр unavailable болсон байдал нь нийт request-ийн амжилтад нөлөөлж, availability буурахад хүргэсэн болохыг туршилтын үр дүнгээр харуулж байна.

---

## 18. Deliberate FAIL Test

SLO threshold system үнэхээр failure илрүүлж байгаа эсэхийг шалгахын тулд тусдаа:

```
slo-test-fail.js
```

файл үүсгэсэн.

Үндсэн SLO: `/report p(95) < 450 ms`

харин deliberate FAIL test дээр зориудаар: `/report p(95) < 100 ms`

болгон өөрчилсөн.

`/report` endpoint нь 200–400 ms random delay-тэй учраас энэ threshold нь илүү хатуу нөхцөл үүсгэсэн.

---

## 19. Deliberate FAIL Results

**`/cart/add`**
- p(95) = 1.78 ms
- Threshold = p(95) < 200 ms
- Result = **PASS**

**`/report`**
- p(95) = 389.88 ms
- Threshold = p(95) < 100 ms
- Result = **FAIL**

k6 дараах threshold failure-ийг мэдээлсэн:

```
thresholds on metrics 'http_req_duration{name:report}' have been crossed
```

**`/pay`**
- Error rate = 3.67%
- Threshold = error rate < 8%
- Result = **PASS**

**Checks**
- Checks = 98.77%
- Threshold = rate > 90%
- Result = **PASS**

**Overall**
- Total HTTP requests = 5547
- Iterations = 1849
- Maximum VUs = 20
- Duration = 2 minutes
- Overall p(95) = 371.42 ms

k6 process exit code: `99`

Ингэснээр deliberate FAIL test нь зориуд өөрчилсөн `/report` threshold-ийг зөв илрүүлсэн.

---

## 20. FAIL Screenshot

FAIL test-ийн screenshot:

```
screenshots/fail.png
```

---

## 21. Test Results Summary

| Test  | Endpoint / Metric | Actual  | Threshold | Result |
|-------|--------------------|---------|-----------|--------|
| PASS  | `/cart/add` p95     | 1.78 ms | < 200 ms  | PASS   |
| PASS  | `/report` p95       | 392.48 ms | < 450 ms | PASS  |
| PASS  | `/pay` error rate    | 5.50%   | < 8%      | PASS   |
| PASS  | Checks              | 98.16%  | > 90%     | PASS   |
| CHAOS | Availability        | 84.21%  | >= 90%    | FAIL   |
| CHAOS | `/cart/add` p95      | 1.52 ms | < 200 ms  | PASS   |
| CHAOS | `/report` p95        | 390.97 ms | < 450 ms | PASS  |
| CHAOS | `/pay` error rate     | 18.36%  | < 8%      | FAIL   |
| CHAOS | Checks               | 84.21%  | > 90%     | FAIL   |
| FAIL  | `/cart/add` p95       | 1.78 ms | < 200 ms  | PASS   |
| FAIL  | `/report` p95         | 389.88 ms | < 100 ms | FAIL  |
| FAIL  | `/pay` error rate      | 3.67%   | < 8%      | PASS   |
| FAIL  | Checks                | 98.77%  | > 90%     | PASS   |

---

## 22. Project Structure

```
lab3-scenarioSLO/
│
├── node_modules/
│
├── results/
│   ├── pass.txt
│   ├── chaos.txt
│   └── fail.txt
│
├── screenshots/
│   ├── pass.png
│   ├── chaos.png
│   └── fail.png
│
├── server.js
├── slo-test.js
├── slo-test-fail.js
├── package.json
├── package-lock.json
├── .gitignore
└── README.md
```

`node_modules/` нь `.gitignore`-оор Git repository-д ороогүй.

---

## 23. Ашигласан командууд

**Project үүсгэх**

```bash
mkdir lab3-scenarioSLO
cd lab3-scenarioSLO
mkdir results
npm init -y
npm install express
```

**Server ажиллуулах**

```bash
node server.js
```

**PASS test**

```bash
k6 run slo-test.js
```

**PASS result хадгалах**

```bash
k6 run slo-test.js 2>&1 | tee results/pass.txt
```

**Chaos test**

```bash
k6 run slo-test.js 2>&1 | tee results/chaos.txt
```

Chaos test-ийн үед серверийг түр зогсоож, 10 секундын дараа дахин ажиллуулсан.

**Deliberate FAIL test**

```bash
k6 run slo-test-fail.js > results/fail.txt 2>&1
echo $?
```

FAIL test-ийн k6 exit code: `99`

---

## 24. Git Commit History

Лабораторийн хөгжүүлэлтийн явцад дараах meaningful commits хийсэн.

```
e8afdd7 feat: create express api for slo lab
2b39b61 Add load testing script for cart, report, and payment endpoints using k6
7cbacc0 test: add chaos availability test results
b400e23 test: add deliberate slo threshold failure
91c740b docs: add fail test screenshot
```

Git repository нь: https://github.com/Hirito572/lab3-scenarioSLO

---

## 25. Үр дүнгийн дүгнэлт

Энэхүү лабораторийн ажлаар чанарын шаардлагыг чанарын сценари болгон тодорхойлж, тэдгээрийг хэмжигдэхүйц SLO threshold болгон хөрвүүлсэн. `/cart/add` endpoint-ийн performance туршилтаар p95 нь 1.78 ms байсан нь 200 ms-ийн threshold-оос бага байв. `/report` endpoint-ийн үндсэн SLO болох p95 < 450 ms нөхцөл мөн PASS болсон бөгөөд бодит p95 нь 392.48 ms байсан. `/pay` endpoint-ийн үндсэн PASS test-ийн error rate 5.50% байсан нь 8%-ийн threshold дотор байв. Chaos test-ийн үед нийт 5733 request-ээс 4828 нь амжилттай болж, request-based availability 84.21% болсон тул 90%-ийн availability SLO хангагдаагүй. Chaos test-ийн үед `/pay` endpoint-ийн error rate 18.36% болж өссөн нь reliability-д нөлөөлсөн. Deliberate FAIL test-ийн үед `/report` endpoint-ийн threshold-ийг p95 < 100 ms болгон зориудаар өөрчилж, бодит p95 389.88 ms гарсан тул k6 threshold failure-ийг зөв илрүүлсэн. Мөн k6 deliberate FAIL test-ийн үед exit code 99 буцаасан. Эдгээр туршилтуудаар performance, reliability болон availability-ийн SLO-уудыг k6 threshold ашиглан хэмжиж, PASS болон FAIL нөхцөлүүдийг ялган баталгаажуулсан.

---

## 26. Optional AI Usage

Энэхүү лабораторийн ажлын явцад AI хэрэгслийг ашиглан k6 threshold-ийн бүтэц, README-ийн зохион байгуулалт болон туршилтын үр дүнг тайлбарлахад туслалцаа авсан. Харин API код, туршилтын execution болон бодит үр дүнг локал орчинд өөрөө ажиллуулж шалгасан. README-д оруулсан performance, reliability болон availability-ийн тоон утгууд нь бодитоор ажиллуулсан k6 test-ийн үр дүн дээр үндэслэсэн.

---

## 27. Conclusion

Лаборатори №3-аар чанарын сценариос SLO, SLO-оос k6 threshold хүртэл шаардлагыг хэмжигдэхүйц хэлбэрт шилжүүлэх үйл явцыг хэрэгжүүлсэн. Express API дээр performance, reliability болон availability гэсэн гурван өөр чанарын шинжийг туршсан. `/cart/add` endpoint нь performance threshold-ийг хангаж, маш бага p95 response time үзүүлсэн. `/report` endpoint нь 200–400 ms random delay-тэй боловч үндсэн 450 ms p95 SLO-г хангасан. `/pay` endpoint-ийн PASS test дээр error rate нь 8%-ийн SLO дотор байсан. Chaos test нь серверийн түр зогсолт request-based availability болон reliability-д хэрхэн нөлөөлж байгааг харуулсан. Энэ туршилтын үед availability 84.21% болж 90%-ийн SLO-оос доогуур гарсан. Deliberate FAIL test-ээр threshold систем нь зориуд тавьсан хатуу нөхцөлийг зөв илрүүлж байгааг баталгаажуулсан. Иймээс k6 threshold нь системийн чанарын шаардлагыг автомат байдлаар шалгах, PASS/FAIL нөхцөлийг тодорхойлох практик арга болохыг туршилтаар харуулсан.