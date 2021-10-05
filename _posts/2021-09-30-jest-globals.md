---
# permalink: "Posts"
layout: single

title: 'Jest - globals'
excerpt: 'Testing Framework handbook'

author_profile: false

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5 # 투명도

data: 2021-09-30 08:40:00 +0900
last_modified_at: 2021-10-06 21:22:00 +0900

categories:
  - post
tags:
  - Jest
  - Unit
  - Test
---

# JEST 메뉴얼 (v 27.2)

---

## Globals

### afterAll(fn, timeout)

- ***

## Globals

beforeAll / afterAll : describe 범위 안에서 전후 동작
beforeEach / afterEach : describe 범위 안의 test 단위 전후로 동작, 외부 describe에 있는 beforeEach는 내부 describe beforeEach보다 먼저 실행된다.

### afterAll(fn, timeout)

- 파일 안의 모든 test가 완료 후 동작하는 함수이다, 함수가 promise 또는 generator를 반환한다면 JEST는 진행하기 전에 promise로부터 resolve되기를 기다릴 것이다. 이 함수는 여러 테스트 간에 전역 변수를 초기화되길 원한다면 종종 유용할 것이다.
- timeout: 선택적으로 당신은 대기시간을 줄 수 있다. (milliseconds, default 5 seconds)
- 만약 이 함수가 descirbe 안에 있다면 describe가 끝날 때 작동할 것이다. 모든 test가 끝난 후가 아닌 test 각각 끝난 후 작동되도록 하려면 'afterEach'함수를 사용하라.

```
const globalDatabase = makeGlobalDatabase();

function cleanUpDatabase(db) {
  db.cleanUp();
}

afterAll(() => {
  cleanUpDatabase(globalDatabase);
});

test('can find things', () => {
  return globalDatabase.find('thing', {}, results => {
    expect(results.length).toBeGreaterThan(0);
  });
});

test('can insert a thing', () => {
  return globalDatabase.insert('thing', makeThing(), response => {
    expect(response.success).toBeTruthy();
  });
});
```

### afterEach(fn, timeout)
