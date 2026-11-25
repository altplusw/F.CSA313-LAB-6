# Лаборатори 6 — Category-Partition ба PICT

**Д.Бүрэнтөгс · B222270092**

## A. Шилжүүлгийн тестийн дизайн

Тестлэх функц нь `POST /transfer`. Үлдэгдэл, үлдсэн өдрийн лимит болон хүсэлтийн дүнг эх дансны валютаар харьцуулна. Жишээнд данс MNT, хөрвүүлэлтийн ханш 1 USD = 3500 MNT гэж тогтоов.

| Сонголт | Төлөөлөх утга ба тэмдэглэгээ |
|---|---|
| From | **Active** ажиллаж буй данс [FROM_EXISTS], **Frozen** царцаасан [ERROR, FROM_EXISTS], **Missing** байхгүй [ERROR] |
| To | **Other** өөр данс, **Missing** байхгүй [ERROR], **SameAccount** эх данстай ижил [ERROR, IF FROM_EXISTS] |
| Amount | **Normal** 0-ээс их, хоёр дээд хязгаараас бага; **ExactBalance** үлдэгдэлтэй тэнцүү, лимитээс бага; **ExactLimit** лимиттэй тэнцүү, үлдэгдлээс бага [SINGLE]; **Zero** 0 [ERROR]; **OverBalance** дансны үлдэгдлээс хэтэрсэн [ERROR]; **OverLimit** лимитээс хэтэрсэн [ERROR] |
| Currency | **Same** MNT → MNT, **Converted** USD → MNT |
| Далд орчин | Эх дансны үлдэгдэл, ашигласан өдрийн лимит, тогтмол ханш. Эдгээрийг мөр бүрийн setup-д тохируулна. |

[ERROR] утгуудыг гар хүснэгтэд тусад нь шалгавал алдааны шалтгаан тодорхой байна. [SINGLE] нь яг үлдсэн лимиттэй тэнцэх тохиолдлыг гар хүснэгтэд нэг удаа тусгайлан шалгана. PICT-д ExactLimit-ийг энгийн эерэг утга болгосноор валют зэрэг хүчин зүйлтэй хослох боломжтой. [IF] нь байхгүй эх дансыг мөн хүлээн авагч гэж тэмдэглэхээс сэргийлнэ. Аль нэг данс байхгүй үед мөнгөн дүнгийн өөрчлөлт тухайн шалгалтад нэмэлт мэдээлэл өгөхгүй тул Amount-д Normal төлөөлөгчийг үлдээнэ.

Хязгаарлалтгүй олонлог 3 × 3 × 6 × 2 = **108**. Эх данс байхгүй үед To-д 2 сонголт, Amount-д 1 сонголт үлдэнэ. Эх данс байгаа хоёр төлөв тус бүрд To байхгүй бол 1 дүн, бусад хоёр To сонголт бүрд 6 дүн байна. Валютын хоёр сонголтыг тооцвол (2 + 2 × (1 + 2 × 6)) × 2 = **56**, нийт **108 → 56**.

Олон алдаанд `NO_FROM → FROZEN → NO_TO → SAME_ACCOUNT → BAD_AMOUNT → INSUFFICIENT → LIMIT` дарааллыг тестийн oracle-д сонгов. Эх данс хүчинтэй бол хүлээн авагчийг шалгаж, хоёр данс зөв үед хөрвүүлсэн дүнг харьцуулах дараалал нь шалтгааныг тогтвортой тайлбарлана. Тодорхойлолт дарааллыг заагаагүй тул энэ нь тестийн дизайны таамаглал юм.

| № | From | To | Amount | Currency | Хүлээгдэх үр дүн |
|---:|---|---|---|---|---|
| 1 | Active | Other | Normal | Same | 201 OK |
| 2 | Missing | Other | Normal | Same | 200 ERROR_NO_FROM |
| 3 | Frozen | Other | Normal | Same | 200 ERROR_FROZEN |
| 4 | Active | Missing | Normal | Same | 200 ERROR_NO_TO |
| 5 | Active | SameAccount | Normal | Same | 200 ERROR_SAME_ACCOUNT |
| 6 | Active | Other | Zero | Same | 200 ERROR_BAD_AMOUNT |
| 7 | Active | Other | OverBalance | Same | 200 ERROR_INSUFFICIENT |
| 8 | Active | Other | OverLimit | Converted | 200 ERROR_LIMIT |
| 9 | Active | Other | ExactLimit [SINGLE] | Same | 201 OK |
| 10 | Active | Other | ExactBalance | Converted | 201 OK |
| 11 | Frozen | SameAccount | OverLimit | Converted | 200 ERROR_FROZEN |

## B. PICT

PICT-ийг Windows дээр CMake, Visual Studio C++ ашиглан эх кодоос бүтээв. [Эх кодын хувилбар ба хэрэглээний гаралт](results/pict-build.txt).

```text
pict browser.model
pict browser.model /o:3
pict transfer-nc.model
pict transfer.model
pict transfer-neg.model
```

