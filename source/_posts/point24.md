---
title: JavaScript 24 点游戏算法
date: 2016-10-14 14:22:09
tags: algorithm
---

24 点：棋牌类益智游戏，要求 4 个数字运算结果等于 24。这个游戏用扑克牌很容易开展，拿一副牌，抽去大小王后以及 J/Q/K 后，剩下 1～10 这 40 张牌(以下用 1 代替 A)。任意抽取4张牌(称为牌组)，用加、减、乘、除(可加括号，高级玩家也可用乘方开方运算)把牌面上的数算成 24。每张牌必须且只能用一次。比如抽出的牌是 3、8、8、9，那么算式可以为 (9-8)×8×3=24。

<!-- more -->

下面用 js 实现任意 4 个整数的 24 算法：

```
function point24(input){

    var index = [[0,1,2,3],[0,1,3,2],[0,2,1,3],
                 [0,2,3,1],[0,3,1,2],[0,3,2,1],
                 [1,2,0,3],[1,2,3,0],[1,3,0,2],
                 [1,3,2,0],[2,3,0,1],[2,3,1,0]],
        times = index.length;

    var success = false,
        finalArr = [],
        originArr = [],	
        is31 = false,
        is22 = false,
        temp31Arr = [],
        res = "";

    function twoRes(arr){
        if (success && !arguments[2]){
            return [];
        }
        var res0 = arr[0] + arr[1],
            res1 = arr[0] - arr[1],
            res2 = arr[0] * arr[1],
            res3 = arr[0] / arr[1],
            res4 = arr[1] - arr[0],
            res5 = arr[1] / arr[0],
            isN = false,
            num = arguments[2] || 24;
        
        isN = isInArr(num,[res0,res1,res2,res3,res4,res5]);
        if ((isN !== false) && arguments[1]){
            switch (isN){
                case 0: 
                    if (!arguments[2]){
                        finalArr = [arr[0],arr[1]];
                    } else {
                        return [arr[0] + "+" + arr[1],num];
                    }
                    break;
                case 1: 
                    if (!arguments[2]){
                        finalArr = [arr[0],arr[1]];
                    } else {
                        return [arr[0] + "-" + arr[1],num];
                    }
                    break;
                case 2: 
                    if (!arguments[2]){
                        finalArr = [arr[0],arr[1]];
                    } else {
                        return [arr[0] + "*" + arr[1],num];
                    }
                    break;
                case 3: 
                    if (!arguments[2]){
                        finalArr = [arr[0],arr[1]];
                    } else {
                        return [arr[0] + "/" + arr[1],num];
                    }
                    break;
                case 4: 
                    if (!arguments[2]){
                        finalArr = [arr[0],arr[1]];
                    } else {
                        return [arr[1] + "-" + arr[0],num];
                    }
                    break;
                case 5: 
                    if (!arguments[2]){
                        finalArr = [arr[0],arr[1]];
                    } else {
                        return [arr[1] + "/" + arr[0],num];
                    }
                    break;
                default :
            }
            return [[res0,res1,res2,res3,res4,res5][isN]];
        }
        return [res0,res1,res2,res3,res4,res5];
    }
    
    function threeRes(arr){
        var arrRes = [],
            len = 0,
            res = [],
            temp = [],
            n = arguments[1];
        
        arrRes = (n === undefined ? twoRes([arr[0],arr[1]]) : twoRes([arr[0],arr[1]],true,true));
        len = arrRes.length;

        for (var i = 0;i < len;i++){
            temp = (n === undefined ? twoRes([arrRes[i],arr[2]]) : twoRes([arrRes[i],arr[2]],true,true));
            res = res.concat(temp);

            if (n !== undefined){
                if (isInArr(n,temp) !== false){
                    temp31Arr = [arrRes[i],arr[2]];
                    return res;
                }
            }
        }
        return res;
    }

    function fourRes1(arr){
        var arrRes = threeRes([arr[0],arr[1],arr[2]]),
            len = arrRes.length,
            res = [],
            temp = [];
        
        for (var i = 0;i < len;i++){
            temp = twoRes([arrRes[i],arr[3]],true);
            res = res.concat(temp);
            if (isInArr(24,temp) !== false){
                success = true;
                is31 = true;
                return res;
            }
        }
        return res;
    }

    function fourRes2(arr){
        var arr1 = twoRes([arr[0],arr[1]]),
            len1 = arr1.length,
            arr2 = twoRes([arr[2],arr[3]]),
            len2 = arr2.length,
            res = [],
            temp = [];
        
        for (var i = 0;i < len1;i++){
            for (var j = 0;j < len2;j++){
                temp = twoRes([arr1[i],arr2[j]],true);
                res = res.concat(temp);
                if (isInArr(24,temp) !== false){
                    success = true;
                    is22 = true;
                    return res;
                }
            }
        }
        return res;
    }

    function fourRes(arr){
        var res1 = [],
            res2 = [],
            res = [];
        
        res1 = fourRes1([arr[0],arr[1],arr[2],arr[3]]);
        res2 = fourRes2([arr[0],arr[1],arr[2],arr[3]]);
        res = res1.concat(res2);
        return res;
    }

    function isInArr(n,arr){
        var len = arr.length;
        for (var i = 0;i < len;i++){
            if (n === arr[i]){
                return i;
            }
        }
        return false;
    }

    function get24(){
        var num0 = 0,num1 = 0,num2 = 0,num3 = 0;
        for (var i = 0;i < times;i++){
            originArr = [input[index[i][0]],input[index[i][1]],input[index[i][2]],input[index[i][3]]]
            if (isInArr(24,fourRes(originArr)) !== false){
                return;
            }
        }
        return;
    }

    get24();

    if (is22){
        var tempArr1 = originArr.slice(0,2),
            tempArr2 = originArr.slice(2,4),
            tempRes1 = "",
            tempRes2 = "",
            operator = "";
            

        tempRes1 = twoRes(tempArr1,true,finalArr[0])[0];
        tempRes2 = twoRes(tempArr2,true,finalArr[1])[0];
        
        if (finalArr[0] >= 0){
            operator = twoRes(finalArr,true,24)[0].match(/[^\d\.\s]/)[0];
        } else {
            operator = twoRes(finalArr,true,24)[0].match(/[^\d\.\s]/g)[1]
        }

        if (operator == "-"){
            if ((finalArr[0] - finalArr[1]) === 24){
                res = "(" + tempRes1 + ")-" + "(" + tempRes2 + ") =24";
            } else {
                res = "(" + tempRes2 + ")-" + "(" + tempRes1 + ") =24";
            }
        } else if (operator == "/"){
            if ((finalArr[0] / finalArr[1]) === 24){
                res = "(" + tempRes1 + ")/" + "(" + tempRes2 + ") =24";
            } else {
                res = "(" + tempRes2 + ")/" + "(" + tempRes1 + ") =24";
            }
        } else {
            res = "(" + tempRes1 + ")" + operator + "(" + tempRes2 + ") =24";
        }
    }

    if (is31){
        var tempArr1 = originArr.slice(0,3),
            tempArr2 = originArr.slice(3,4),
            tempArr3 = originArr.slice(0,2),
            tempRes1 = "",
            operator1 = "",
            operator2 = "";

        threeRes(tempArr1,finalArr[0]);
        tempRes1 = twoRes(tempArr3,true,temp31Arr[0])[0];
        if (temp31Arr[0] >= 0){
            operator1 = twoRes([temp31Arr[0],temp31Arr[1]],true,finalArr[0])[0].match(/[^\d\.\s]/)[0];
        } else {
            operator1 = twoRes([temp31Arr[0],temp31Arr[1]],true,finalArr[0])[0].match(/[^\d\.\s]/g)[1];
        }
        
        if (operator1 == "-"){
            if ((temp31Arr[0] - temp31Arr[1]) === finalArr[0]){
                res = "(" + tempRes1 + ")-"  + temp31Arr[1];
            } else {
                res = temp31Arr[1] + "-(" + tempRes1 + ")";
            }
        } else if (operator1 == "/"){
            if ((temp31Arr[0] / temp31Arr[1]) === finalArr[0]){
                res = "(" + tempRes1 + ")/"  + temp31Arr[1];
            } else {
                res = temp31Arr[1] + "/(" + tempRes1 + ")";
            }
        } else {
            res = "(" + tempRes1 + ")" + operator1  + temp31Arr[1];
        }

        if (finalArr[0] >= 0){
            operator2 = twoRes(finalArr,true,24)[0].match(/[^\d\.\s]/)[0];
        } else {
            operator2 = twoRes(finalArr,true,24)[0].match(/[^\d\.\s]/g)[1];
        }
        if (operator2 == "-"){
            if (finalArr[0] - finalArr[1] === 24){
                res = "(" + res + ")-" + finalArr[1];
            } else {
                res = finalArr[1] + "-(" + res + ")";
            }
        } else if (operator2 == "/"){
            if (finalArr[0] / finalArr[1] === 24){
                res = "(" + res + ")/" + finalArr[1];
            } else {
                res = finalArr[1] + "/(" + res + ")";
            }
        } else {
            res = "(" + res + ")" + operator2 + finalArr[1] + "=24";
        }
    }
    return res;
}
```

以上 point24 方法的实参是一个长度为 4 的一维数组，例如：

```
point24([1,2,3,4]);

// 结果：
// "((1+2)+3)*4=24"
```
