---
layout: default
title: Asyncronous in JavaScript
parent: JavaScript
nav_order: 3
---

JavaScript의 비동기적인 특성에 대해서 정리한다.

---
# 리소스들
* Medium - How JavaScript works_ Event loop and the rise of Async programming
  + https://www.howdy-mj.me/javascript/asynchronous-programming (한국어 번역본)


---
JavaScript는 싱글 스레드로 동작하지만, Event Loop를 이용해서 멀티스레드로 작업이 가능하다. 

브라우저에서 Event Loop를 제공해주며, 자세한 내용은 구글의 V8 스펙을 참조하자

[How JavaScript works: inside the V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)

콜 스택에서 바로 큐로 넘어가는게 아니라 중간에 Web APIs를 한 번 거쳐 큐로 넘어간다. 이는 어떤 함수나 이벤트가 종료될 때까지 시간이 오래 걸릴 수 있기 때문에, 자바스크립트 엔진이 직접 처리하는 것이 아니라 브라우저에 위임한다.

큐는 크게 태스크 큐(Task Queue), 마이크로 태스크 큐(Micro Task Queue), rAF 큐(Request Animation Frame Queue)로 나뉜다.

 * 태스크 큐(Task Queue): `setTimeout()`, `setInterval()`과 같은 비동기 함수의 콜백 함수 또는 이벤트 핸들러가 대기하는 곳이다.
 * 마이크로 태스크 큐(Micro Task Queue): `Promise()`의 후속 처리 메서드의 콜백 함수나 `MutationObserver()`가 대기하는 곳이다.
 * rAF 큐(Request Animation Frame Queue): `requestAnimationFrame()`처럼 애니메이션을 업데이트하는 콜백 함수가 대기하는 곳이다.

각 큐에 대한 실행 우선 순위는 마이크로 태스크 큐 > rAF 큐 > 태스트 큐 순서이다. 이벤트 루프는 해당 순서대로 대기하고 있는 함수들을 보고 있다가 차례대로 콜 스택에 가져와 실행한다.

---
비동기 구문은 크게 아래와 같다.
 * 콜백 함수
 * Promise
 * async/await

# 콜백(Callback) 함수

콜백은 다른 함수의 인자로 함수를 넘기는 것을 말한다. 콜백 함수로 비동기 프로그래밍을 짤 수 있지만, 모든 콜백 함수가 비동기이진 않다. 예를 들어 map(), filter()의 첫 번째 인자로 들어가는 콜백 함수는 동기식으로 호출된다.

```javascript
// 해당 코드는 [자바스크립트 Deep Dive, 이웅모 (2020)]의 프로미스(p842)에서 가져왔습니다.

const POSTS_URL = 'https://jsonplaceholder.typicode.com/posts';

const getPosts = (url) => {
  const xhr = new XMLHttpRequest();
  xhr.open('GET', url);
  xhr.send();

  xhr.onload = () => {
    if (xhr.status === 200) {
      console.log(JSON.parse(xhr.response));
    } else {
      console.error(`${xhr.status} ${xhr.statusText}`);
    }
  };
};

const posts = getPosts(POSTS_URL);
console.log('posts: ', posts);
```

여기서 post를 console로 찍었을 때 어떤 결과가 나올까?

xhr.onload()가 비동기로 동작하기 때문에 post는 undefined라는 결과를 반환한다. 이렇듯 비동기로 동작하는 함수는 외부에서 그 값을 바로 참조하지 못하여, 무조건 콜백 함수 내부에서 그 처리를 진행해야 한다.


```javascript
const getPosts = (url, whenSuccess, whenFail) => {
  const xhr = new XMLHttpRequest();
  xhr.open('GET', url);
  xhr.send();

  xhr.onload = () => {
    if (xhr.status === 200) {
      whenSuccess(JSON.parse(xhr.response));
    } else {
      whenFail(xhr.status, xhr.statusText);
    }
  };
};

const handlePosts = (response) => {
  // ...
};

const errorHandling = (status, statusText) => {
  // ...
};

const posts = getPosts(POSTS_URL, handlePosts, errorHandling);
```

따라서 콜백의 후속 처리를 모두 그 콜백 함수 내에서 처리해야 하기 때문에, 위처럼 다시 콜백함수를 넘기는 수 밖에 없게 되었다.

그런데 만약 해당 콜백 함수에 또 예외 처리를 해야 하거나, 여러 에러 상황에 각기 다른 조치를 취해야 한다면 어떻게 해야할까? 끔찍하게도 이것 역시 또 다른 콜백함수로 넘겨야 한다. 그리고 이런 상황이 곧 **'콜백 헬(callback hell)'**이라는 단어로 불러졌다.


### callback 함수와 반복문
map(), filter(), reduce()함수의 인자로 제공되는 콜백 함수는 놀랍게도! 동기식으로 호출된다.

map filter callback <= 이렇게 검색하면 여러 관련글을 찾아볼 수 있다.







---
# Promise
https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise/Promise


 * promise 관련 링크
   * https://joshua1988.github.io/web-development/javascript/promise-for-beginners/


### promise의 응용
 * node.js에서 mysql 라이브러리는 callback 형식으로 작동한다. 쿼리문을 수행 후, callback을 통해 처리하는 방식이다.
 * 예를들어 회원가입을 할 때, 먼저 중복된 정보가 있는 지 검사를 해야 하며, 중복 된 정보가 없다면 데이터베이스에 회원 정보를 추가하는 예제 코드이다.

```javascript
const mysql = require('mysql')
// variable setting
const con = mysql.createConnection({ /* DB info */ })
// mysql 실행구문
const sql1 = `SELECT count(*) cnt FROM member where id = 'test' and pw = '1234'`
const sql2 = `INSERT INTO member SET id = 'test', pw = '1234'`
con.query(sql1, (err, res) => {
  if (res.rows[0] === 0) {
      con.query(sql2, (err, res) => {
        if (err) console.log(err)
        else console.log('회원정보가 추가되었습니다.')
      })
  } else {
    console.log('이미 중복된 회원 정보가 존재합니다.')
  }
})
```

 * 위의 사례는 쿼리문을 2개 중복시켰을 경우 실행되는 예제 입니다. 하지만 쿼리문이 2개가 아니라 3개, 4개, 5개 이상이며 심지어 else 구문에도 쿼리문이 존재한다면 어떤 일이 발생할까요?
```javascript
con.query(sql1, (err1, res1) => {
  if (!err1) {
    con.query(sql2, (err2, res2) => {
      if (!err2) {
        con.query(sql3, (err3, res3) => {
          if (!err3) {
            con.query(sql4, (err4, res4) => {
              if (!err4) {
                con.query(sql5, (err4, res4) => { /* ... */ })
              }
            })
          }
        })
      }
    })
  }
})
```
 * 위 예제는 코드가 중첩되어서 매우 읽기 어려워진다.

 * 하지만 promise를 사용하면, 특히 거기다가 async/await을 사용하면 아래처럼 깔끔하게 표현 가능하다.
```javascript
const queryExec = sql => new Promise ((resolve, reject) => {
  con.query(sql, function (err, res) => {
    err ? reject(err) : resolve(res) // reject는 예외 처리를 할 때 사용합니다.
  })
});
(async () => {
  try {
    const res1 = await queryExec(sql1)
    const res2 = await queryExec(sql2)
    const res3 = await queryExec(sql3)
    const res4 = await queryExec(sql4)
    const res5 = await queryExec(sql5)
    /* ... 최종 처리 코드 작성 ... */
  } catch (err) {
    console.log(err)
  }
})()
```

 * 매우 깔끔하게 변경되었다. 그러나, 이 코드에도 문제가 있습니다. 여기서 필요한 것은 res1 ~ res5 까지의 데이터이다. 하지만 res1을 가져온 다음에 res2를 가져오고, res3을 가져오고 등의 방식으로 실행되기 때문에 시간이 낭비된다.
 * 즉, 최종 코드가 처리 되기 까지 걸리는 시간은 모든 쿼리문이 실행 시간의 합과 동일하다.
 * Query Time 1 + Query Time 2 + ... + Query Time n = 최종 실행 까지 걸리는 시간


 * 이럴 때 Promise.All을 사용하면 된다.
```javascript
(async () => {
  try {
    Promise
      .all([queryExec(sql1), queryExec(sql2), queryExec(sql3), queryExec(sql4), queryExec(sql5)])
      .then(values => {
        const [res1, res2, res3, res4, res5] = values
        /* ... 최종 처리 코드 작성 ... */
      })
  } catch (err) {
    console.log(err)
  }
})()
```

 * Promise.all은 모든 Promise 객체들 중 가장 마지막으로 끝나는 것을 기준으로 then 구문을 처리한다.
 * 따라서 쿼리문 5개 중 가장 늦게 끝나는 것을 기준으로 실행 되게 된다.
 * Max(QueryTime1, QueryTime2, ... , QueryTime5) = 최종 실행 가지 걸리는 시간


### Promise의 에러처리

* 1.then()의 두 번째 인자로 에러를 처리하는 방법
```javascript
getData().then(
  handleSuccess,
  handleError
);
```

* 2.catch()를 이용하는 방법
```javascript
getData().then().catch();
```

위 2가지 방법 모두 프로미스의 reject() 메서드가 호출되어 실패 상태가 된 경우에 실행된다. 간단하게 말해서 프로미스의 로직이 정상적으로 돌아가지 않는 경우는 아래처럼 호출된다.

```javascript
function getData() {
  return new Promise(function(resolve, reject) {
    reject('failed');
  });
}

// 1. then()의 두 번째 인자로 콜백함수를 넘겨서 에러를 처리하는 코드
getData().then(function() {
  // ...
}, function(err) {
  console.log(err);
});

// 2. catch()로 에러를 처리하는 코드
getData().then().catch(function(err) {
  console.log(err);
});
```

 * Promise의 에러처리는 가급적 catch를 이용해야 한다 <= then()의 2번째 인자로는 감지 못하는 오류가 있기 때문에

```javascript
// then()의 두 번째 인자로는 감지하지 못하는 오류
function getData() {
  return new Promise(function(resolve, reject) {
    resolve('hi');
  });
}

getData().then(function(result) {
  console.log(result);
  throw new Error("Error in then()"); // Uncaught (in promise) Error: Error in then()
}, function(err) {
  console.log('then error : ', err);
});
```
위 코드는 then의 첫번째 인자의 함수에서 발생하는 throw new로 에러가 나는 것을 처리하지 못한다.

따라서 아래처럼 catch로 작성하는게 좋다.
```javascript
// catch()로 오류를 감지하는 코드
function getData() {
  return new Promise(function(resolve, reject) {
    resolve('hi');
  });
}

getData().then(function(result) {
  console.log(result); // hi
  throw new Error("Error in then()");
}).catch(function(err) {
  console.log('then error : ', err); // then error :  Error: Error in then()
});
```

##### Promise 의 같단한 예

```javascript
function startGame() {
  let counter = 0;

  document.querySelector('button').addEventListener('click', () => {
	++counter;
  });

  return new Promise((resolve, reject) => {
	setTimeout(() => {
	  if (counter > 5) {
		resolve();  // <= 성공하면 resolve
	  } else {
		reject();   // <= 실패하면 reject
	  }
	}, 2000)
  });
}

startGame()
  .then(() => alert('You win!'))
  .catch(() => alert('You lost!'));
```




---
# async/await
Promise는 여전히 콜백 함수를 사용하기 때문에, Callback Hell 처럼 코드가 중첩되어지는 문제가 발생할 수 밖에 없다.

ES8에서 보다 간단하고 가독성 좋게 비동기 처리를 동기 처리처럼 구현하는 async/await가 도입되었다.

async/await는 Promise를 기반으로 동작하며, **then/catch/finally와 같은 후속 처리 메서드 없이 마치 동기 처리처럼 사용할 수 있다.**



```javascript
const getPostWithAsync = async (url) => {
  try {
    const response = await fetch(url);
    return await response.json(); // 혹은 다른 형태로 데이터 전처리 가능
  } catch (err) {
    console.err(err);
  } finally {
    console.log('끝');
  }
};

const posts = getPostWithAsync(POSTS_URL);
console.log('posts: ', posts);

posts.then(console.log);
```
콜백 함수나 Promise는 무조건 api를 호출한 후, 또 다른 콜백 함수를 실행하여 데이터의 처리가 가능했지만, async/await는 해당 함수 내부에서 바로 동기 처리처럼 데이터를 수정할 수 있다. 

또한 try/catch 문으로 에러 처리도 훨씬 수월하다.



---
# async (axio)

### axios
 * 3rd party package
 * install
   * npm install --save axio


---
# fetch
 * 최신 브라우저에 추가된 API이다.
 * 하지만, 작성하는데 여러가지를 신경써야될게 많아서 많이는 안쓰이는 듯 하다.


---
# 동기 함수 vs 비동기 함수
JavaScript에서 비동기 함수와 동기 함수를 구분하는 것은 중요한데, 때로는 이를 판별하기가 어려울 수 있습니다. 일반적으로 다음과 같은 방법으로 구분할 수 있습니다:

* 1. 콜백 함수 유무:
비동기 함수는 종종 콜백 함수를 인자로 받습니다. 이 콜백 함수는 비동기 작업이 완료되었을 때 호출됩니다.
동기 함수는 콜백 함수를 사용하지 않습니다. 대신, 함수 호출이 완료될 때까지 프로그램의 실행이 일시 중지됩니다.

* 2. Promise 또는 async/await 사용 여부:

비동기 함수는 종종 Promise나 async/await를 사용하여 비동기적으로 작동합니다.
동기 함수는 일반적으로 이러한 비동기적인 기능을 사용하지 않습니다.

* 3. 이벤트 기반 함수:
비동기 함수는 종종 이벤트 기반입니다. 즉, 특정 이벤트가 발생할 때만 실행됩니다.
동기 함수는 주로 순차적으로 실행됩니다.

* 4. 함수 이름과 문서화:
함수의 이름과 주석을 통해 해당 함수가 비동기적인지 동기적인지 알 수 있습니다. 예를 들어, 함수 이름이 "loadData"인 경우 비동기적일 가능성이 높고, "calculateSum"과 같은 함수는 동기적일 가능성이 높습니다.


그러나 때로는 함수의 특성에 대한 명확한 정보가 없거나, 코드를 작성한 개발자가 문서화를 제대로 하지 않았을 수 있으므로 이를 판별하는 데 어려움이 있을 수 있습니다. 이러한 경우에는 함수의 구현을 자세히 살펴보고, 외부 리소스와의 상호 작용이 있는지, 콜백 함수 또는 Promise가 있는지 확인하는 것이 도움이 될 수 있습니다.

