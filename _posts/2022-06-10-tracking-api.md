---
layout: post
title: 배송 API 테스트
date: 2022-06-10
categories: [ETC]
tag: [html, ajax]
---

오늘은 택배 배송조회를 이야기 하려한다. 회사에서 준비 중인 서비스에서 택배 배송조회를 구현해야 해서 택배 배송조회에 대해서 알아보고 직접 API 테스트를 간단히 해보았다. 

테스트 코드는 HTML, AJAX를 통해서 구현했다. API 서버로 https://tracker.delivery/guide 이 곳을 이용했다. 

```html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" type="text/css" href="./memo.css" />
    <title>TEST</title>
</head>
<body>
    <div class="Box">
        <div class="InputBox">
            <h3>배송 API TEST</h3>
            <form id="searchForm" class="InputForm">
                <input id="trackNum" type="number" placeholder="송장번호">
                <select id="carrierList"></select>
                <button id="searchButton" type="button">조회</button>
            </form>
        </div>
        <div class="OutputBox">
            <textarea id="responseData" cols="20" rows="10"></textarea>
        </div>
    </div>
    <script src="https://code.jquery.com/jquery-3.4.1.js"></script>
    <script>
        const selectBox = document.getElementById("carrierList");
        const responseData = document.getElementById("responseData");

        $.ajax({
            type: "get",
            url: "https://apis.tracker.delivery/carriers",
            data: "",
            dataType: 'json',
            success: function(data) {
                console.log(data);
                for(let i = 0; i < data.length; i++){
                    let carrierData = document.createElement("option");
                    carrierData.text = data[i].name;
                    carrierData.value = data[i].id;
                    selectBox.options.add(carrierData);    
                }    
            },
            error: function(request, status, error) {
                console.log("code:" + request.status + "\n" + "message:" + request.responseText + "\n" + "error:" + error);
            }
        });


        $(function(){
            $('#searchButton').on("click",function(){
                let selectedCarrier = document.getElementById("carrierList").value;
                let trackNum = document.getElementById("trackNum").value;
                console.log(selectedCarrier);
                console.log(trackNum);

                $.ajax({
                    type: "get",
                    url: "https://apis.tracker.delivery/carriers/" + selectedCarrier + "/tracks/" + trackNum,
                    data: "",
                    dataType: 'json',
                    success: function(data) {
                        console.log(data);
                        let string = '';
                        string += "From:" + JSON.stringify(data.from.name) + "\r\n"; 
                        string += "State:" + JSON.stringify(data.state.text) + "\r\n"; 
                        string += "To:" + JSON.stringify(data.to.name) + "\r\n"; 
                        
                        console.log(string);
                        responseData.innerHTML = string;
                    },
                    error: function(request, status, error) {
                        console.log("code:" + request.status + "\n" + "message:" + request.responseText + "\n" + "error:" + error);
                    }
                });
            });
        });
    </script>
</body>
</html>

```

처음에는 몇 가지 택배사만 직접 구현을 해보려 했다. 하지만 생각보다 많이 귀찮았다. 직접 구현을 한다면 해당 택배사의 API 요청, 개발자 가입, CORS 등등 이슈가 좀 있었다. 그래서 무료로 사용할 수 있는 API 서버를 통해 테스트를 진행했다.

테스트는 무료 API 서버를 이용했지만 서비스에 적용한다면 ***스마트택배 API*** 라는 좋은 유료 서비스가 있다. 유료 서비스니 서버 안정성이나 개발 중 기술지원에 유리하지 않을까? 싶다. 
