# gllvm-parse

refer to this [github issue](https://github.com/SRI-CSL/gllvm/issues/63) for the background information


go to where `gllvm` is installed (for me it's `/Users/peterchan/go/src/github.com/SRI-CSL/gllvm`), add the following to `shared/parser.go`

right below line 171

```go
"-mthumb": {0, pr.assembleOnlyCallback},
```

right below line 382

```go
{`^-mcpu=.+$`, flagInfo{0, pr.compileUnaryCallback}},
{`^-mfpu=.+$`, flagInfo{0, pr.compileUnaryCallback}},
{`^-mfloat-abi=.+$`, flagInfo{0, pr.compileUnaryCallback}},
{`^-specs=.+$`, flagInfo{0, pr.compileUnaryCallback}},
```

right below line 399

```go
{`^--target=.+$`, flagInfo{0, pr.compileUnaryCallback}},
```

---

To re-build gllvm, run `go install github.com/SRI-CSL/gllvm/cmd/...`

After the above step, `gllvm` is updated amd should recognize all the new options