+++
title = "Floating Point Hell"
date = "2014-03-16T13:33:19-05:00"
highlightjslanguages = ["go"]
+++

This blog post will show how to deal with floating point error in JavaScript by encoding all uint64's, and int64's to strings in the Json Marshaling.
<!--more-->

JavaScript has 53-bits of integer precision, this is a problem when trying to represent a 64-bit number. Ways of solving this in JavaScript is to use [bigint](https://v8project.blogspot.com/2018/05/bigint.html) or [math.js](http://mathjs.org/), but when parsing Json we can't use this.

---

# When it goes wrong

This is a example of how it can go wrong. Go encodes the Json properly, but when parsed by JavaScript the number does not match.

## Go

Base data structure

```go
type LargeId uint64

type Data struct {
    Id      LargeId
    BigNum int64
}
```

Encode and Decode data

```go
data := Data{Id: 12345678901234567892, BigNum: 12000000000002539}
output, _ := json.Marshal(data)
fmt.Println("Data -> Json", string(output))
input := string(output)
output2 := Data{}
json.Unmarshal([]byte(input), &output2)
fmt.Printf("Json -> Data %+v\n", output2)

//Output
Data -> Json {"Id":12345678901234567892,"BigNum":12000000000002539}
Json -> Data {Id:12345678901234567892 BigNum:12000000000002539}
```

As you can see Go has no problem encoding and decoding the large integers.
But lets see what happens when JavaScript tries to decode the same integers.

## JavaScript

```javascript
var data = '{"Id":12345678901234567892,"BigNum":12000000000002539}'
console.log(JSON.parse(data))

//Output
Object {Id: 12345678901234567000, BigNum: 12000000000002540}
```

JavaScript ended up decoding the number, but its wrong.
But if we check for equivalence in JavaScript it returns true.

```javascript
> 12000000000002539 === 12000000000002540
true
```

---

# How to fix it

The easiest way to fix this is to encode the int64 to a string, that way when parsed in JavaScript the number is properly represented.
Below I have added `json:",string"` tag to the end of every int64, this tells the `encoding/json` package to encode that field as a string instead of a integer.

## Go

Base data structure with tags

```go
type LargeId uint64

type Data struct {
    Id      LargeId `json:",string"`
    BigNum int64  `json:",string"`
}
```

Encode and Decode data

```go
data := Data{Id: 12345678901234567892, BigNum: 12000000000002539}
output, _ := json.Marshal(data)
fmt.Println("Data -> Json", string(output))
input := string(output)
output2 := Data{}
json.Unmarshal([]byte(input), &output2)
fmt.Printf("Json -> Data %+v\n", output2)

//Output
Data -> Json {"Id":"12345678901234567892","BigNum":"12000000000002539"}
Json -> Data {Id:12345678901234567892 BigNum:12000000000002539}
```

With the tag we can encode and decode the Json and keep the int64 type internally.

## JavaScript

```javascript
var data = '{"Id":"12345678901234567892","BigNum":"12000000000002539"}'
console.log(JSON.parse(data))

//Output
Object {Id: "12345678901234567892", BigNum: "12000000000002539"}
```

Now in JavaScript the number is properly represented because they are strings.