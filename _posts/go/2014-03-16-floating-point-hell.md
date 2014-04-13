---
layout: post
category : go
title: Floating Point Hell
tags : [go, javascript, json]
---
{% include JB/setup %}

This blog post will show how to deal with floating point error in JavaScript by encoding all uint64's, and int64's to strings in the Json Marshaling.
JavaScript has 53-bits of integer precision, this is a problem when trying to represent a 64-bit number.
Ways of solving this in JavaScript is to use [bigint](http://www.khjk.org/log/2010/nov/jsbigint.html) or [math.Long](http://docs.closure-library.googlecode.com/git/class_goog_math_Long.html), but when parsing Json we can't use this.

---

# When it goes wrong
This is a example of how it can go wrong.
Go encodes the Json properly, but when parsed by JavaScript the number does not match.

## Go
Base data structure
{% highlight go %}
type LargeId uint64

type Data struct {
    Id      LargeId
    BigNum int64
}
{% endhighlight %}
Encode and Decode data
{% highlight go %}
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
{% endhighlight %}

As you can see Go has no problem encoding and decoding the large integers.
But lets see what happens when JavaScript tries to decode the same integers.

## JavaScript
{% highlight javascript %}
var data = '{"Id":12345678901234567892,"BigNum":12000000000002539}'
console.log(JSON.parse(data))

//Output
Object {Id: 12345678901234567000, BigNum: 12000000000002540}
{% endhighlight %}

JavaScript ended up decoding the number, but its wrong.
But if we check for equivalence in JavaScript it returns true.

{% highlight javascript %}
> 12000000000002539 === 12000000000002540
true
{% endhighlight %}


---

# How to fix it
The easiest way to fix this is to encode the int64 to a string, that way when parsed in JavaScript the number is properly represented.
Below I have added `json:",string"` tag to the end of every int64, this tells the `encoding/json` package to encode that field as a string instead of a integer.

## Go
Base data structure with tags
{% highlight go %}
type LargeId uint64

type Data struct {
    Id      LargeId `json:",string"`
    BigNum int64  `json:",string"`
}
{% endhighlight %}
Encode and Decode data

{% highlight go %}
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
{% endhighlight %}

With the tag we can encode and decode the Json and keep the int64 type internally.

## JavaScript
{% highlight javascript %}
var data = '{"Id":"12345678901234567892","BigNum":"12000000000002539"}'
console.log(JSON.parse(data))

//Output
Object {Id: "12345678901234567892", BigNum: "12000000000002539"}
{% endhighlight %}
Now in JavaScript the number is properly represented because they are strings.
