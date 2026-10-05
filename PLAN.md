# PLAN — thaiutils-go

Status: **M0 scaffolded** · Owner: @Chawn · Last updated: 2026-10-06

## Goal
The go-to, dependency-free Go toolkit for Thai-specific data handling that every Thai
backend ends up re-writing: national ID validation, baht-to-text, Buddhist-era dates,
Thai phone numbers and Thai digits. Success = other Go modules import it (dependents on
pkg.go.dev / GitHub "Used by"), not stars.

## Non-goals
- Thai word segmentation / NLP (separate, much bigger problem).
- Address data (see the `thai-address` repo).
- PromptPay (see `promptpay-go`).

## Design principles
1. **Zero dependencies** outside the standard library. Ever.
2. One small sub-package per concern so users import only what they need:
   `github.com/Chawn/thaiutils-go/thaiid`, `/bahttext`, `/thaidate`, `/thaiphone`, `/thainum`.
3. Pure functions, no global state, no panics on user input — return `error`.
4. Every exported identifier has a doc comment and at least one `Example` test (these
   render on pkg.go.dev and are the #1 adoption driver).
5. Support the two latest Go releases + Go 1.22 (CI matrix).

## API design (target for v0.1.0)

### `thaiid` — Thai national ID / tax ID (13 digits)
```go
func Validate(id string) error           // ErrLength, ErrNonDigit, ErrChecksum
func IsValid(id string) bool
func Normalize(id string) (string, error) // strips spaces/dashes, converts Thai digits ๐-๙
func Format(id string) (string, error)    // "1-2345-67890-12-1"
func CheckDigit(first12 string) (int, error)
func Type(id string) (PersonType, error)  // first digit category (1-8, 0) — document the official meaning per digit, cite source
```
Checksum: `sum = Σ d[i]*(13-i) for i in 0..11`; `check = (11 - sum%11) % 10`.
The same algorithm validates 13-digit juristic-person tax IDs (starting with 0).

### `bahttext` — amount to Thai words
```go
func Format(amount float64) (string, error)           // convenience; rounds half-up to 2 dp
func FormatDecimalString(s string) (string, error)    // exact: "1234567.89" — no float error
func FormatSatang(satang int64) string                // exact integer path, used by both above
```
Rules:
- `0` → `ศูนย์บาทถ้วน` (the reference npm lib returns `""` — we deliberately differ; document it).
- Unit-position 1 after another digit → `เอ็ด` (`101` → `หนึ่งร้อยเอ็ด`, `1001` → `หนึ่งพันเอ็ด`, `1000001` → `หนึ่งล้านเอ็ด`).
- Tens: `10` → `สิบ`, `20` → `ยี่สิบ`. Repeat per ล้าน group (`21000021` → `ยี่สิบเอ็ดล้านยี่สิบเอ็ด`).
- Satang part uses the same rules; integer amount ends with `ถ้วน`.
- Negative → prefix `ลบ`. Values beyond int64 satang → error.
- Option (v0.2): `WithoutEd()` for orgs that write `หนึ่งร้อยหนึ่ง`.

Test vectors: `testdata/bahttext_vectors.json` (generated from npm `thai-baht-text@2.0.5`,
except the `0` case noted above). Port **all** of them into a table test.

### `thaidate` — Buddhist Era dates
```go
const BEOffset = 543
func ToBE(year int) int
func FromBE(year int) int
func Format(t time.Time, layout string) string // Go-style layout + tokens: Thai month full/abbr, Thai weekday, BE year
func Parse(layout, value string) (time.Time, error)
var MonthsFull, MonthsShort, WeekdaysFull, WeekdaysShort [..]string
func ThaiDigits(s string) string // optional rendering with ๐-๙
```
Must handle: `6 ตุลาคม 2569`, `6 ต.ค. 69`, `06/10/2569`, Thai digits. Bangkok tz helper:
`var Bangkok = time.FixedZone("ICT", 7*3600)` (no tzdata dependency).

### `thaiphone` — Thai phone numbers
```go
func Normalize(s string) (string, error) // → E.164 "+66812345678"
func FormatLocal(e164 string) string     // "081-234-5678" / "02-123-4567"
func Kind(s string) (Kind, error)        // Mobile | Landline | TollFree | Unknown
```
Accept `0812345678`, `+66 81 234 5678`, `66812345678`, `081-234-5678`.

### `thainum` — Thai digits
```go
func ToArabic(s string) string  // "๑๒๓" → "123"
func ToThai(s string) string    // "123" → "๑๒๓"
```

## Milestones
- [x] **M0 — Scaffold**: module, CI, license, plan, stub packages compiling.
- [ ] **M1 — thaiid + thainum** (smallest, highest-value). Full tests, examples, fuzz test for `Validate` (no panic).
- [ ] **M2 — bahttext**: integer-satang core, all vectors pass, fuzz test (no panic, deterministic).
- [ ] **M3 — thaidate**: format/parse round-trip tests, Thai-digit variants.
- [ ] **M4 — thaiphone**: normalization table tests incl. bad inputs.
- [ ] **M5 — Release v0.1.0**: README examples verified, `go doc` complete, tag, `GOPROXY=proxy.golang.org go list -m github.com/Chawn/thaiutils-go@v0.1.0` to index on pkg.go.dev.
- [ ] **M6 — Adoption**: announce in Thai dev communities (Golang Thailand), then open a PR to `avelino/awesome-go` once coverage ≥ 80% and Go Report Card is A+. Use it in Ben's own company projects first — real dependents count.

## Definition of done (every milestone)
- `go vet ./...` and `go test -race ./...` pass; coverage ≥ 90% for the package.
- golangci-lint clean.
- README section for the package updated with a runnable example.
- PLAN.md checkbox ticked with a one-line note.

## Open questions (ask Ben before deciding)
- Rounding policy for `bahttext.Format(float64)`: half-up (proposed) vs banker's.
- Should `thaiid.Type` ship in v0.1 (needs an authoritative source for digit meanings)?
