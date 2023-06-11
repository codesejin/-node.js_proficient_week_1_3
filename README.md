# node.js_proficient_week_1_3
node.js 숙련주차 -jwt

# JWT가 무엇인가요?
- 1) 간략한 정리!
    - JSON 형태의 데이터를 안전하게 교환하여 사용할 수 있게 해줍니다.
    - 인터넷 표준으로서 자리잡은 규격입니다.
    - 여러가지 암호화 알고리즘을 사용할 수 있습니다.
    - `header.payload.signature` 의 형식으로 3가지의 데이터를 포함합니다. (개미처럼 머리, 가슴, 배)
    때문에 JWT 형식으로 변환 된 데이터는 항상 2개의 `.` 이 포함된 데이터여야 합니다.
- 2) 어떻게 생긴건가요?
    
   ![image](https://github.com/codesejin/node.js_proficient_week_1_3/assets/101460733/3c4bffac-ca26-4250-87e6-530ae7125232)

    
    https://jwt.io/ 에서 간단히 확인할 수 있는데요, 위에서 말했듯이 개미처럼 머리, 가슴, 배와 같은 3가지를 가졌습니다.
    
    - **header**(머리)는 signature(배)에서 어떤 암호화를 사용하여 생성된 데이터인지 표현합니다.
    - **payload**(가슴)는 개발자가 원하는 데이터를 저장합니다.
    - **signature**(배)는 이 토큰이 변조되지 않은 정상적인 토큰인지 확인할 수 있게 도와줍니다.
- 3) 더 알아두어야 할 특성
    - JWT는 **비밀 키**를 모르더라도 **복호화(Decode)** 가 가능합니다.
    변조만 불가능 할 뿐, **누구나 복호화**하여 보는것은 가능하다는 의미가 됩니다!
    - 때문에 **민감한 정보(개인정보, 비밀번호 등)** 는 담지 않도록 해야합니다.
    - 특정 언어에서만 사용 가능한것은 아닙니다!
    단지 개념으로서 존재하고, 이 개념을 코드로 구현하여 공개된 코드를 우리가 사용하는게 일반적입니다.
- 4) 쿠키, 세션과 어떻게 다른가요?
    
    데이터를 교환하고 **관리하는 방식** 인 쿠키/세션과 달리, **JWT는** 단순히 **데이터를 표현하는 형식** 입니다.
    
    - JWT로 만든 데이터를 브라우저로 보내도 쿠키처럼 자동으로 저장되지는 않지만, 변조가 거의 불가능하고 서버에 데이터를 저장하지 않기 때문에 서버를 **Stateless(무상태)** 로 관리할 수 있기 때문에 최근 많이 쓰이는 기술중 하나입니다.
    - **Stateless(무상태)** 와 **Stateful(상태 보존)** 의 차이를 간단히 설명하자면,
    Node.js 서버가 언제든 죽었다 살아나도 **똑같은 동작**을 하면 **Stateless**하다고 볼 수 있습니다.
    반대로 서버가 죽었다 살아났을때 조금이라도 **동작이 다른** 경우 **Stateful**하다고 볼 수 있겠죠.
    - 서버가 **스스로** 어떤 기억을 갖고 다른 결정을 하냐 마냐의 차이라고 보면 더 쉽습니다 😉
    - 로그인 정보를 서버에 저장하게 되면 무조건 **Stateful**(상태 보존)이라고 볼 수 있죠!

---
```npm init -y```
```npm install jsonwebtoken```: 오픈소스 라이브러리
---



# jwt 생성, 복호화, 검증

```
const jwt = require("jsonwebtoken");

const payloadData = {
    myPayloadData : 1234
}

// jwt 생성
const token = jwt.sign(payloadData, "mysecretKey");
console.log(token);

// jwt의 payload 데이터를 복호화
const decodedValue = jwt.decode(token);
console.log("복호한 token 입니다." , decodedValue);

// jwt를 만들었을 때, 사용하나 비밀키가 일치하는지 검증
const decodedValueByVerify = jwt.verify(token, "mysecretKey");
console.log("decodedValueByVerify :", decodedValueByVerify);

// jwt를 만들었을 때, 사용하나 비밀키가 일치하는지 검증하지만 에러 발생
const decodedValueByVerifyToError = jwt.verify(token, "비밀키를 다르게 입력해봄.");
console.log("decodedValueByVerifyToError :", decodedValueByVerifyToError);
```

# JWT를 왜 써야 하는건가요?
    
JWT는 아래와 같은 특징을 가지고 있습니다.

1. JWT가 **인증 서버** 에서 발급되었는지 **위변조 여부를 확인** 할 수 있습니다.
2. **누구든지** JWT 내부에 들어있는 정보를 확인할 수 있습니다. **(복호화)**

만약 JWT를 사용하지 않은 상태에서 사용자 로그인을 구현하려고 하면 어떻게 될까요?

# jwt를 적용하지 않은 로그인 api

- 사용자의 정보가 `sparta` 이름을 가진 쿠키에 할당됩니다.
- 쿠키의 속성값이나 만료 시간을 클라이언트가 언제든지 **수정**할 수 있습니다.
- 쿠키의 **위변조 여부**를 확인 할 수 없습니다.

```
const express = require('express');
const app = express();

app.post('/login', function (req, res, next) {
  const user = { // 사용자 정보
    userId: 203, // 사용자의 고유 아이디 (Primary key)
    email: "archepro84@gmail.com", // 사용자의 이메일
    name: "이용우", // 사용자의 이름
  }

  res.cookie('sparta', user);  // sparta 라는 이름을 가진 쿠키에 user 객체를 할당합니다.
  return res.status(200).end();
});

app.listen(5002, () => {
  console.log(5002, "번호로 서버가 켜졌어요!");
});
```
sparta라는 쿠키 전달 - value에 있는 값은 http문자를 표현하는 방식

- 사용자의 userId, email, name이 바로 노출되는 문제점
브라우저에서 해당 값을 수정할 수 있는데 만약에 특정 클라이언트가 브로우저에 들어가서 쿠키 값을 수정하게 되면 서버에서는 변경된 데이터로 인식하고 문제가 발생할 수 있음

<img width="1076" alt="image" src="https://github.com/codesejin/node.js_proficient_week_1_3/assets/101460733/2b10a6c9-3281-4ff7-a0b2-8a037005ff6b">


# jwt를 적용한 로그인 api

- 사용자의 정보를 Payload에 저장한 JWT를 `sparta` 이름을 가진 쿠키에 할당됩니다.
- JWT를 생성할 때 위변조 여부를 확인할 수 있는 **비밀키**를 사용하였습니다.
- 쿠키의 만료시간과 별개로 **JWT의 만료시간**을 설정하였습니다.

```
const express = require('express');
const JWT = require("jsonwebtoken");
const app = express();

app.post('/login', async (req, res) => {
  // 사용자 정보
  const user = {
    userId: 203,
    email: "archepro84@gmail.com",
    name: "이용우",
  }

  // 사용자 정보를 JWT로 생성
  const userJWT = await JWT.sign(user, // user 변수의 데이터를 payload에 할당
    "secretOrPrivateKey", // JWT의 비밀키를 secretOrPrivateKey라는 문자열로 할당
    { expiresIn: "1h" } // JWT의 인증 만료시간을 1시간으로 설정
  );

  // userJWT 변수를 sparta 라는 이름을 가진 쿠키에 Bearer 토큰 형식으로 할당
  res.cookie('sparta', `Bearer ${userJWT}`);
  return res.status(200).end();
});

app.listen(5002, () => {
  console.log(5002, "번호로 서버가 켜졌어요!");
});
```

<img width="1092" alt="image" src="https://github.com/codesejin/node.js_proficient_week_1_3/assets/101460733/6498d056-1dbd-4122-b34b-fa8be65f960a">


# 이 암호화 된 데이터는 어떻게 쓸 수 있나요?
  - 보통 암호화 된 데이터는 클라이언트(브라우저)가 전달받아 다양한 수단(**쿠키**, **로컬스토리지** 등)을 통해 저장하여 API 서버에 요청을 할 때 서버가 요구하는 HTTP 인증 양식에 맞게 보내주어 인증을 시도합니다!
  - 비유하자면, 놀이공원의 자유이용권과 비슷한거죠!
      - **회원가입**: 회원권 구매
      - **로그인**: 회원권으로 놀이공원 입장
      - **로그인 확인**: 놀이기구 탑승 전마다 유효한 회원권인지 확인
      - **내 정보 조회**: 내 회원권이 목에 잘 걸려 있는지 확인하고, 내 이름과 사진, 바코드 확인
