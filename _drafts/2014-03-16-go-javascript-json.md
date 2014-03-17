---
layout: post
category : go
title: Floating Point Hell
tags : [go, javascript, json]
---
{% include JB/setup %}


JavaScript has 53-bits of precision, this is a problem when trying to represent a 64-bit number. 

---
# When it goes wrong

## Go
Base Data Structure

    type LargeId uint64

    type Data struct {
        Id      LargeId
        LargNum uint64
        Num     int
    }

Encode and Decode data

    data := Data{Id: 12345678901234567892, LargNum: 12000000000002539, Num: 12}
    output, _ := json.Marshal(data)
    fmt.Println("Data -> Json", string(output))
    input := string(output)
    output2 := Data{}
    json.Unmarshal([]byte(input), &output2)
    fmt.Printf("Json -> Data %+v\n", output2)

    //Output
    Data -> Json {"Id":12345678901234567892,"Number":12000000000002539,"Int":12}
    Json -> Data {Id:12345678901234567892 Number:12000000000002539 Int:12}


## JavaScript

    var data = '{"Id":12345678901234567892,"Number":12000000000002539,"Int":12}'
    console.log(JSON.parse(data))

    //Output
    Object {Id: 12345678901234567000, Number: 12000000000002540, Int: 12}

As you can see precision error


---
# How to fix it

## Go

Fix it on the Go side

    type Data struct {
        Id      LargeId `json:",string"`
        LargNum uint64  `json:",string"`
        Num     int
    }

Same base Data structure but with tags, this way go can use the uint64 type internaly

Encode and Decode data

    data := Data2{Id: 12345678901234567892, LargNum: 12000000000002539, Num: 12}
    output, _ := json.Marshal(data)
    fmt.Println("Data -> Json", string(output))
    input := string(output)
    output2 := Data{}
    json.Unmarshal([]byte(input), &output2)
    fmt.Printf("Json -> Data %+v\n", output2)

    //Output
    Data -> Json {"Id":"12345678901234567892","Number":"12000000000002539","Int":12}
    Json -> Data {Id:12345678901234567892 Number:12000000000002539 Int:12}

As you can see when doing from the json it decodes back into normal ints

## JavaScript

    var data = '{"Id":"12345678901234567892","Number":"12000000000002539","Int":12}'
    console.log(JSON.parse(data))

    //Output
    Object {Id: "12345678901234567892", Number: "12000000000002539", Int: 12}

Because it keeps them as strings there is no precision error




    