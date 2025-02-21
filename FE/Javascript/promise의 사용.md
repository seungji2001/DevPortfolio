# 비동기 프로그래밍

### async함수
- 프로미스를 기반으로 동작
```javascript
function getMult10Promise (number) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(number * 10);
    }, 1000);
  });
}
```

```javascript
async function doAsyncWorks () {
  const result1 = await getMult10Promise(1);
  console.log(result1);

  const result2 = await getMult10Promise(2);
  console.log(result2);

  const result3 = await getMult10Promise(3);
  console.log(result3);
}

doAsyncWorks();
console.log('💡 이 문구가 먼저 출력됨');
```

```javascript
const DEADLINE = 1400;

function getRelayPromise(name, start, failMsg){
    return new Promise((resolve, reject) => {
        const time = 1000 + Math.random() * 500;

        setTimeout(() => {
            if(time < DEADLINE){
                console.log(`${name} 도착 - ${(start + time)/1000} 초 `);
                resolve(start + time);
            }else{
                console.log(failMsg);
                reject((start+time)/1000);
            }
        }, time);
    })
}


async function relay5(){
    try{
        const time1 = await getRelayPromise('철수', 0, '철수부터 광탈입니다..');

        const time2 = await getRelayPromise('영희', time1, '영희가 완주하지 못했네요');
    }catch(msg){
        console.log(`완주실패 ${msg}`)
    }finally{
        console.log('경기 종료')
    }
}
```

## 네트워크에서 활용 방법
```javascript
fetch('url')
.then(response => {
    console.log(response);
    return response;
})
.then(response => response.json())
.then(console.log)
```

- 반환되는 결과
- 요청의 결과에 대한 정보들을 담은 객체를 json 객체로 반환하여 반환

### 프로미스 형태로 구현시
```javascript
fetch(url)
.then(result => result.json())
.then(arr =>{
    return arr.sort((a, b) => {
        return a.record - b.record
    })[0].runner_idx
})
.then(winnerIdx => {
    return fetch(`url/${winnerIdx}`)
})
.then(result => result.json())
.then(({school_idx}) => school_idx)
.then(schoolIdx=>{
    return fetch(`url/${schoolIdx}`)
})
.then(result => result.json())
.then(console.log)
.catch(console.error)
```

### async, await으로 구현

```javascript
async function getWinnersSchool(){
    const raceResult = await fetch(url)
    .then(result => result.json);

    const winnerIdx = raceResult
    .sort((a, b) => {
        return a.record - b.record
    })[0].runner_idx;

    const winnerInfo = await fetch(`url/${winnerIdx}`)
    .then(result => result.json());

    const schoolIdx = winnerInfo.school_idx;

    const schoolInfo = await fetch(`url/${schoolIdx}`)
    .then(result => result.json());

    console.log(shcoolInfo);
}
```

getWinnersSchool();
