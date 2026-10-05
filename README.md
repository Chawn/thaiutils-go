# thaiutils-go

> **Status: pre-alpha — under active development. APIs will change until v0.1.0.**
> See [PLAN.md](PLAN.md) for the roadmap.

Dependency-free Go helpers for Thai data: national ID, baht text, Buddhist-era dates, phone numbers, Thai digits.

ชุดเครื่องมือ Go สำหรับข้อมูลแบบไทย: ตรวจเลขบัตรประชาชน, แปลงจำนวนเงินเป็นคำอ่าน (บาทถ้วน), วันที่ พ.ศ., เบอร์โทรไทย, เลขไทย — ไม่มี dependency

## Install
```sh
go get github.com/Chawn/thaiutils-go
```

## Usage (target API)
```go
import (
    "github.com/Chawn/thaiutils-go/bahttext"
    "github.com/Chawn/thaiutils-go/thaiid"
)

thaiid.IsValid("1101700230708")          // true / false
s, _ := bahttext.FormatSatang(123456789) // "หนึ่งล้านสองแสนสามหมื่นสี่พันห้าร้อยหกสิบเจ็ดบาทแปดสิบเก้าสตางค์"
```

## Development
```sh
go vet ./...
go test -race ./...
golangci-lint run   # if installed
```

## Contributing
Issues and PRs welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). Thai or English.

## License
MIT © Chawn and contributors
