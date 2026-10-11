**Оюутны нэр:** Б.Хишигцогзол
**Оюутны код:** B222270834

## Орчин

```
$ node -v
v18.19.0
$ newman -v
6.2.2
```

## Даалгавар 2: Тест дизайн

**Бие даан тестлэгдэх функц:** `POST /registrations` — оюутныг хичээлд бүртгэх.

### Сонголт ба төлөөлөх утгууд

| Сонголт | Төлөөлөх утга | Серверт яаж бий болгох |
|---|---|---|
| studentID-гийн хүчинтэй байдал | идэвхтэй | PUT /students/ID `{"status":"active"}` |
| | идэвхгүй | PUT /students/ID `{"status":"inactive"}` |
| | байхгүй | хэзээ ч PUT хийгдээгүй ID |
| Оюутны үзсэн хичээлүүд | урьдач нөхцөлийг хангана | coursesTaken-д шаардлагатай бүх хичээл бий |
| | хангахгүй | coursesTaken-д шаардлагатай хичээл дутуу |
| courseID-гийн хүчинтэй байдал | байгаа | PUT /courses/ID хийсэн |
| | байхгүй | хэзээ ч PUT хийгдээгүй ID |
| Хичээлийн урьдач нөхцөл | бүгд үзсэн | урьдач бүгд coursesTaken-д бий |
| | огт үзээгүй | урьдач нэг нь ч coursesTaken-д алга |
| | зарим нь | хоёр урьдачаас нэгийг нь л үзсэн |

### Спецификаци

| # | Тестийн нэр | Оюутан | Хичээл | Статус | result |
|---|---|---|---|---|---|
| 1 | Идэвхтэй оюутан урьдач нөхцөл хангасан бол бүртгэгдэнэ | B231234567: active, [CS201] | CS313: урьдач [CS201] | 201 | OK |
| 2 | Байхгүй оюутан бүртгүүлэхэд ERROR_NO_STUDENT | B000000001: үүсгэхгүй | CS313: урьдач [CS201] | 200 | ERROR_NO_STUDENT |
| 3 | Идэвхгүй оюутан бүртгүүлэхэд ERROR_INACTIVE_STUDENT | B000000002: inactive, [CS201] | CS313: урьдач [CS201] | 200 | ERROR_INACTIVE_STUDENT |
| 4 | Байхгүй хичээлд бүртгүүлэхэд ERROR_NO_COURSE | B000000003: active, [CS201] | CS999: үүсгэхгүй | 200 | ERROR_NO_COURSE |
| 5 | Урьдач нөхцөл огт үзээгүй бол ERROR_PREREQUISITES | B000000004: active, [CS101] | CS313: урьдач [CS201] | 200 | ERROR_PREREQUISITES |
| 6 | Идэвхгүй оюутан байхгүй хичээлд бүртгүүлэхэд ERROR_INACTIVE_STUDENT | B000000005: inactive, [CS201] | CS998: үүсгэхгүй | 200 | ERROR_INACTIVE_STUDENT |
| 7 | Байхгүй оюутан байхгүй хичээлд бүртгүүлэхэд ERROR_NO_STUDENT | B000000006: үүсгэхгүй | CS997: үүсгэхгүй | 200 | ERROR_NO_STUDENT |
| 8 | Урьдачийн зарим нь дутуу бол ERROR_PREREQUISITES | B000000007: active, [CS201] | CS401: урьдач [CS201, CS301] | 200 | ERROR_PREREQUISITES |
| 9 | Урьдач нөхцөлгүй хичээлд хоосон coursesTaken-тэй оюутан бүртгэгдэнэ | B000000008: active, [] | CS100: урьдач [] | 201 | OK |
| 10 | studentID талбар дутуу бол ERROR_BAD_REQUEST | (body-д studentID байхгүй) | CS313 | 400 | ERROR_BAD_REQUEST |

6, 7 нь давхар алдааны хослол (сервер оюутныг хичээлээс өмнө шалгадаг тул оюутны алдаа ирнэ), 9, 10 нь хязгаарын тохиолдол. 


## Даалгавар 3: Collection

`lab05-collection.json` нь 10 бие даасан тесттэй. Тест бүр Postman дээр тусдаа folder (01–10) бөгөөд өөрийн setup PUT хүсэлтүүд, POST /registrations болон 2 oracle-той (статус код + `result` утга). Нийт **24 хүсэлт, 20 assertion**. Тест бүр өөр studentID ашиглаж, өөрийн өгөгдлөө өөрөө бэлддэг тул бусдаасаа хамааралгүй. registrationID-ийн яг утгыг шалгаагүй.

## Даалгавар 4: Newman-аар автоматжуулах

Ашигласан командууд (Git Bash):

```bash
mkdir -p results
newman run lab05-collection.json 2>&1 | tee results/newman-pass.txt; echo "exit=${PIPESTATUS[0]}"
newman run lab05-collection-fail.json 2>&1 | tee results/newman-fail.txt; echo "exit=${PIPESTATUS[0]}"
# серверийг унтраасны дараа:
newman run lab05-collection.json 2>&1 | tee results/newman-down.txt; echo "exit=${PIPESTATUS[0]}"
```

| Ажиллуулалт | Collection | Гаралт | requests (executed / failed) | assertions (executed / failed) | Exit code |
|---|---|---|---|---|---|
| PASS | `lab05-collection.json` | `results/newman-pass.txt` | 24 / 0 | 20 / 0 | **0** |
| FAIL | `lab05-collection-fail.json` | `results/newman-fail.txt` | 24 / 0 | 20 / 1 | **1** |
| DOWN | `lab05-collection.json` (сервер унтраатай) | `results/newman-down.txt` | 24 / 24 | 20 / 20 | **1** |

**Тестийн тоо:** 20 assertion, `results/newman-pass.txt`-ийн «assertions executed» тоотой ижил.

**FAIL:** `lab05-collection-fail.json` нь үндсэн collection-ийн хуулбар бөгөөд зөвхөн «01 Happy path»-ийн статусын oracle-ийг зориуд `201`-ээс `200` болгосон. Newman `expected response to have status code 200 but got 201` гэж 1 assertion унагаж, exit code 1 буцаасан. CI quality gate-ийн зарчмаар exit code 0-ээс өөр бол pipeline зогсож, код цааш нэвтрэхгүй. Үндсэн `lab05-collection.json` зөв хэвээр үлдсэн.

**DOWN:** Серверийг унтраахад бүх 24 хүсэлт `connect ECONNREFUSED 127.0.0.1:3000` алдаа авч, exit code 1 буцаасан. Энэ бол **интерфейсийн алдаа**: сервертэй холболт огт тогтоогдоогүй тул хариу ирээгүй, oracle-ууд шалгах зүйлгүй үлдсэн. Харин FAIL-д сервер зөв хариу (201) өгсөн ч oracle-ийн хүлээлт буруу байсан нь **oracle-ийн алдаа** юм.