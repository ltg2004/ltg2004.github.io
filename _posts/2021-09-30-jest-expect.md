---
permalink: "Posts"
title: "Jest - expect"
excerpt: "Testing Framework handbook"
categories:
  - Blog
tags:
  - Blog
last_modified_at: 2021-09-30T08:40:00
---

# JEST 메뉴얼 (v 27.2)

---

## Export

### expect(value)

- 테스트를 할 때 값이 특정 조건을 충족하는지 확인이 필요한데 Expect는 여러가지 상황을 검증할 수 있는 수많은 'Matcher'에 액세스 할 수 있도록 도와줍니다.

```
test('the best flavor is grapefruit', () => {
  expect(bestLaCroixFlavor()).toBe('grapefruit');
});
```

### expect.extend(matchers)

- 이 함수는 matcher를 구현할 수 있는 함수다. JEST가 제공해주는 matcher중 내가 원하는 matcher가 없는 경우 직접 구현할 수 있게 제공한다.
  예를 들어서, 숫자 범위르 검증하는 테스트가 필요하다고 해보자.
  이 경우, 기본 제공하는 matcher기능(toBeLessThan(), toBeGreatherThan())으로도 테스트를 해볼 수 있지만 2번의 matcher를 호출해야하므로 불편할 수 있다. 그러니 직접 범위를 검증하는 matcher를 만들어보자.
  expectExtend.test.js 파을을 하나 만들고 아래의 matcher 생성 코드를 작성하자.

```
expect.extend({
  toBeWithinRange(received, floor, ceiling) {
    const pass = received >= floor && received <= ceiling;
    if (pass) {
      return {
        message: () =>
          `expected ${received} not to be within range ${floor} - ${ceiling}`,
        pass: true,
      };
    } else {
      return {
        message: () =>
          `expected ${received} to be within range ${floor} - ${ceiling}`,
        pass: false,
      };
    }
  },
});
```

- toBeWithinRange라는 이름으로 숫자의 범위를 검증하는 코드를 구현했다.
  인자로는 총 3개를 받는다.

1. received: expect()의 인자로 넣은 값
2. floor: 최소 범위
3. ceiling: 최대 범위

- 아래는 테스트 코드 예제이다.

```
test('numeric ranges', () => {
  expect(100).toBeWithinRange(90, 110);
  expect(101).not.toBeWithinRange(0, 100);
  expect({apples: 6, bananas: 3}).toEqual({
    apples: expect.toBeWithinRange(1, 10),
    bananas: expect.not.toBeWithinRange(11, 20),
  });
});
```

#### 비동기 Matchers

- 'expect.extend'는 비동기 matchers도 지원한다. 비동기 matchers는 promise를 반환한다. 그래서 반환되는 값을 기다려야 합니다.
- matcher 안에서 비동기 함수를 호출하여 5라는 값을 얻는다. 이 후 expect()로 전달한 값을 5로 나눈다. 그에 대한 결과를 반환한다.

```
expect.extend({
  async toBeDivisibleByExternalValue(received) {
    const externalValue = await getExternalValueFromRemoteSource();
    const pass = received % externalValue == 0;
    if (pass) {
      return {
        message: () =>
          `expected ${received} not to be divisible by ${externalValue}`,
        pass: true,
      };
    } else {
      return {
        message: () =>
          `expected ${received} to be divisible by ${externalValue}`,
        pass: false,
      };
    }
  },
});
function getExternalValueFromRemoteSource() {
    return new Promise((resolve, reject) => {
        resolve(5);
    })
}
```

- 아래는 테스트 코드 예제이다.

```
test('is divisible by external value', async () => {
  await expect(100).toBeDivisibleByExternalValue();
  await expect(101).not.toBeDivisibleByExternalValue();
});
```

#### Custom Matchers API

- Matchers는 두 개의 키를 객체로 반환해야한다. 'pass'는 일치 예상값이랑 일치하거나 다를 경우를 말하며, 'message'는 테스트가 실패했을 경우 에러 메시지를 반환한다.
  'pass'가 false이고 'expect(x).yourMatcher()'이 테스트가 실패일 때 'mesage'는 에러 메시지를 반환해야 한다.
  그리고 'pass'가 true이고 'expect(x).not.yourMatcher()'가 실패일 경우 'message'는 에러 메시지를 반환해야 한다.

```
expect.extend({
  yourMatcher(x, y, z) {
    return {
      pass: true,
      message: () => '',
    };
  },
});
```

##### helper functions

###### this.isNot

-

###### this.promise

- 'rejects': 'matcher'가 promise .rejects modifier로 호출된 경우
- 'resolves': 'matcher'가 promise .resolves modifier로 호출된 경우
- '': 'matcher'가 promise modifier로 호출되지 않은 경우

###### this.equals(a, b)

- 2개의 객체가 재귀적으로 같은 값을 가지고 있다면 true를 반환하는 깊은 비교를 하는 함수이다.

###### this.expand

- Matcher가 확장 옵션으로 호출되었다는 Boolean 속성입니다. '--expand' flag를 붙이고 JEST 실행 시 'this.expend'는 JEST 전체 diff 및 오류를 표시할 것인지 정할 것입니다.

###### this.utils

- 'jest-matcher-utils'에서 나온 'this.utils'안에는 유용한 도구가 많이 있다.
  대표적으로 오류 메시지를 멋지게 형식화하는 'matcherHint', 'printExpected', 'printReceived' 등이 있다.

```
const {diff} = require('jest-diff');
expect.extend({
  toBe(received, expected) {
    const options = {
      comment: 'Object.is equality',
      isNot: this.isNot,
      promise: this.promise,
    };

    const pass = Object.is(received, expected);

    const message = pass
      ? () =>
          this.utils.matcherHint('toBe', undefined, undefined, options) +
          '\n\n' +
          `Expected: not ${this.utils.printExpected(expected)}\n` +
          `Received: ${this.utils.printReceived(received)}`
      : () => {
          const diffString = diff(expected, received, {
            expand: this.expand,
          });
          return (
            this.utils.matcherHint('toBe', undefined, undefined, options) +
            '\n\n' +
            (diffString && diffString.includes('- Expect')
              ? `Difference:\n\n${diffString}`
              : `Expected: ${this.utils.printExpected(expected)}\n` +
                `Received: ${this.utils.printReceived(received)}`)
          );
        };

    return {actual: received, message, pass};
  },
});
```

- 위의 코드가 출력되면 다음과 같습니다.

```
  expect(received).toBe(expected)

    Expected value to be (using Object.is):
      "banana"
    Received:
      "apple"

```

#### Custom snapshot matchers

- 'Custom snapshot matchers'를 사용하려면 'jest-snapshot'을 가져와서 스냅샷 테스팅을 사용할 수 있습니다.

##### toMatchSnapShot

- 여기의 'toMatchSnapShot'matcher는 주어진 길이만큼 문자열을 저장합니다.

```
const {toMatchSnapshot} = require('jest-snapshot');

expect.extend({
  toMatchTrimmedSnapshot(received, length) {
    return toMatchSnapshot.call(
      this,
      received.substring(0, length),
      'toMatchTrimmedSnapshot',
    );
  },
});

it('stores only 10 characters', () => {
  expect('extra long string oh my gerd').toMatchTrimmedSnapshot(10);
});

/*
Stored snapshot will look like:

exports[`stores only 10 characters: toMatchTrimmedSnapshot 1`] = `"extra long"`;
*/
```

##### toMatchInlineSnapshot

- 'toMatchInlineSnapshot'는 인라인 스냅샷을 가능하게 해준다. 이 스냅샷은 정확하게 사용자 정의 matcher에 추가된다.
  However, inline snapshot will always try to append to the first argument or the second when the first argument is the property matcher, so it's not possible to accept custom arguments in the custom matchers.

```
const {toMatchInlineSnapshot} = require('jest-snapshot');

expect.extend({
  toMatchTrimmedInlineSnapshot(received, ...rest) {
    return toMatchInlineSnapshot.call(this, received.substring(0, 10), ...rest);
  },
});

it('stores only 10 characters', () => {
  expect('extra long string oh my gerd').toMatchTrimmedInlineSnapshot();
  /*
  The snapshot will be added inline like
  expect('extra long string oh my gerd').toMatchTrimmedInlineSnapshot(
    `"extra long"`
  );
  */
});
```

#### async

- 'custom inline snapshot matcher'가 비동기인 경우, 즉 async-await를 사용하는 경우 동일한 호출에 대한 동시 처리를 지원하지 않습니다.
- JEST는 스냅샷을 올바르게 업데이트하는데 사용된 'custom inline snapshot matcher'를 찾기 위해 추가적으로 context 정보를 필요로 합니다.

```
const {toMatchInlineSnapshot} = require('jest-snapshot');

expect.extend({
  async toMatchObservationInlineSnapshot(fn, ...rest) {
    // The error (and its stacktrace) must be created before any `await`
    this.error = new Error();

    // The implementation of `observe` doesn't matter.
    // It only matters that the custom snapshot matcher is async.
    const observation = await observe(async () => {
      await fn();
    });

    return toMatchInlineSnapshot.call(this, recording, ...rest);
  },
});

it('observes something', async () => {
  await expect(async () => {
    return 'async action';
  }).toMatchTrimmedInlineSnapshot();
  /*
  The snapshot will be added inline like
  await expect(async () => {
    return 'async action';
  }).toMatchTrimmedInlineSnapshot(`"async action"`);
  */
});
```

#### Bail out

- JEST는 대게 테스트에서 예상되는 모든 스냅샷과 맞추려고 시도합니다.
  때로는 이전 스냅샷이 실패한 경우 테스트를 계속 하는 것이 합리적이지 않을 수 있습니다. 예를 들어 당신이 다양한 transitions 후에 state-machine 스냅샷을 만들 때 한 transition이 잘못된 상태를 생성하면 테스트가 중단될 수도 있습니다.
  아래의 경우는 불일치가 일어나는 모든 스냅샷을 수집하는 것 대신 불일치를 발생시키는 'custom snapshot matcher'를 구현할 수 있습니다.

```
const {toMatchInlineSnapshot} = require('jest-snapshot');

expect.extend({
  toMatchStateInlineSnapshot(...args) {
    this.dontThrow = () => {};

    return toMatchInlineSnapshot.call(this, ...args);
  },
});

let state = 'initial';

function transition() {
  // Typo in the implementation should cause the test to fail
  if (state === 'INITIAL') {
    state = 'pending';
  } else if (state === 'pending') {
    state = 'done';
  }
}

it('transitions as expected', () => {
  expect(state).toMatchStateInlineSnapshot(`"initial"`);

  transition();
  // Already produces a mismatch. No point in continuing the test.
  expect(state).toMatchStateInlineSnapshot(`"loading"`);

  transition();
  expect(state).toMatchStateInlineSnapshot(`"done"`);
});
```

### expect.anything()

- 'ex'ect.anything()'은 뭐든 일치하도록 합니다. 단, null 또는 undefined는 제외입니다.
  리터럴 값 대신 'toEqual' 또는 'toBeCalledWith' 안에서 사용이 가능합니다.
  예를 들어 null이 아닌 인수로 호출된 목업 함수를 확인한다면 아래와 같이 할 수 있다.

```
test('map calls its argument with a non-null argument', () => {
  const mock = jest.fn();
  [1].map(x => mock(x));
  expect(mock).toBeCalledWith(expect.anything());
});
```

### expect.any(constructor)

- 'expect.any(constructor)'는 주어진 생성자로 만들어진 모든 것과 일치하게 합니다.
  리터럴 값 대신 'toEqual' 또는 'toBeCalledWith' 안에서 사용이 가능합니다.
  예를 들어 숫자로 된 목업 함수를 호출한 것을 확인한다면 아래와 같이 할 수 있다.

```
function randocall(fn) {
  return fn(Math.floor(Math.random() * 6 + 1));
}

test('randocall calls its callback with a number', () => {
  const mock = jest.fn();
  randocall(mock);
  expect(mock).toBeCalledWith(expect.any(Number));
});
```

### expect.arrayContaining(array)

- 'expect.arrayContaining(array)'는 예상하는 배열안에 있는 모든 요소를 포함하는 수신된 배열과 일치하게 합니다. 즉, 예상되는 배열은 수신받은 배열의 하위 요소입니다. 그러므로, 예상하는 배열안에 없는 요소도 포함해서 수신된 배열과 일치하게 합니다.
  리터럴 값 대신 'toEqual' 또는 'toBeCalledWith' 안에서 사용이 가능합니다.

```
describe('arrayContaining', () => {
  const expected = ['Alice', 'Bob'];
  it('matches even if received contains additional elements', () => {
    expect(['Alice', 'Bob', 'Eve']).toEqual(expect.arrayContaining(expected));
  });
  it('does not match if received does not contain expected elements', () => {
    expect(['Bob', 'Eve']).not.toEqual(expect.arrayContaining(expected));
  });
});
```

```
describe('Beware of a misunderstanding! A sequence of dice rolls', () => {
  const expected = [1, 2, 3, 4, 5, 6];
  it('matches even with an unexpected number 7', () => {
    expect([4, 1, 6, 7, 3, 5, 2, 5, 4, 6]).toEqual(
      expect.arrayContaining(expected),
    );
  });
  it('does not match without an expected number 2', () => {
    expect([4, 1, 6, 7, 3, 5, 7, 5, 4, 6]).not.toEqual(
      expect.arrayContaining(expected),
    );
  });
});
```

### expect.not.arrayContaining(array)

- 'expect.not.arrayContaining(array)'는 예상하는 배열안에 있는 모든 요소를 포함하지 않는 수신된 배열과 일치하게 합니다. 즉, 예상되는 배열은 수신받은 배열의 하위 요소가 아닙니다. 그러므로, 예상하는 배열안에 없는 요소도 포함해서 수신된 배열과 일치하게 합니다.
- 'expect.arrayContaining(array)'의 반대입니다.

```
describe('not.arrayContaining', () => {
  const expected = ['Samantha'];

  it('matches if the actual array does not contain the expected elements', () => {
    expect(['Alice', 'Bob', 'Eve']).toEqual(
      expect.not.arrayContaining(expected),
    );
  });
});
```

### expect.assertions(number)

- 'expect.assertions(number)'는 테스트하는 동안 특정 수의 주장(assertions)이 호출되었는지를 확인한다.
  이것은 비동기 코드를 테스트할 때 callback의 주장(assertions)이 실제로 호출되었는지 확인하기 위해 사용하는데 종종 유용하게 쓰인다.
  예를 들어 'callback1'과 'callback2' 두 callback 함수를 인수로 받는 'doAsync'함수를 보자, 순서를 알 서없는 상태로 두 함수를 비동기로 호출합니다.

```
test('doAsync calls both callbacks', () => {
  expect.assertions(2);
  function callback1(data) {
    expect(data).toBeTruthy();
  }
  function callback2(data) {
    expect(data).toBeTruthy();
  }

  doAsync(callback1, callback2);
});
```

- 'expect.assertions(2)'는 두 callback함수가 실행되도록 합니다.

### expect.hasAssertions()

- 'expect.hasAssertions()'는 테스트 중에 적어도 하나의 주장(assertion)이 호출되었는지 확인을 한다. 이것은 비동기 코드를 테스트할 때 callback의 주장(assertions)이 실제로 호출되었는지 확인하기 위해 사용하는데 종종 유용하게 쓰인다.
  예를 들어 모든 상태를 다루는 몇개의 함수가 있다고 보자. 'prepareState'는 state 객체로 callback을 호출하고, 'validateState' 해당 state 객체에서 샐행되며, 'waitOnState'는 모든 'prepareState'의 콜백이 완료되기까지 기다렸다가 promise를 반환한다.

```
test('prepareState prepares a valid state', () => {
  expect.hasAssertions();
  prepareState(state => {
    expect(validateState(state)).toBeTruthy();
  });
  return waitOnState();
});
```

- 'expect.hasAssertions()'는 'prepareState' 콜백이 실제로 호출되도록한다.

### expect.not.objectContaining(object)

- 'expect.not.objectContaining(object)'는 예상하는 객체의 속성과 재귀적으로 일치하지 않는 어떤 객체와 일치시킵니다.
  즉, 예상된 객체는 수신한 객체의 하위 집합이 아닙니다. 그러므로, 예상하는 객체에 없는 속성이 포함된 수신된 객체와 일치를 시킵니다.

```
describe('not.objectContaining', () => {
  const expected = {foo: 'bar'};

  it('matches if the actual object does not contain expected key: value pairs', () => {
    expect({bar: 'baz'}).toEqual(expect.not.objectContaining(expected));
  });
});
```

### expect.not.stringContaining(string)

- 'expect.not.stringContaining(string)'는 문자열 타입 아니거나 예상한 문자와 정확하게 일치하지 않는 문자열이라면 인자로 받은 값과 일치하게 한다.

```
describe('not.stringContaining', () => {
  const expected = 'Hello world!';

  it('matches if the received value does not contain the expected substring', () => {
    expect('How are you?').toEqual(expect.not.stringContaining(expected));
  });
});
```

### expect.not.stringMatching(string | regexp)

- 'expect.not.stringMatching(string | regexp)'는 문자열 타입 아니거나 예상한 정규식 패턴과 일치하지 않는 문자열이라면 인자로 받은 값과 일치하게 한다.

```

describe('not.stringMatching', () => {
const expected = /Hello world!/;
    it('matches if the received value does not match the expected regex', () => {
        expect('How are you?').toEqual(expect.not.stringMatching(expected));
    });
});

```

### expect.objectContaining(object)

- 'expect.objectContaining(object)'는 예상되는 객체의 속성이 재귀적으로 일치하는 인자로 들어오는 객체와 일치하게 한다.
  즉, 예상되는 객체는 인자로 받은 객체의 하위 집합입니다. 따라서 예상되는 객체에 있는 속성이 포함된 수신된 객체와 일치하게 합니다.
- 예상 객체의 리터럴 속성 값 대신 matchers, expect.anything() 등을 사용할 수 있습니다.
- 'onPress'함수가 이벤트 객체와 같이 호출된다고 가정해본다면, 이벤트에 'event.x' 및 'event.y' 속성이 있는지 확인하기만 하면 됩니다. 다음과 같이 할 수 있습니다.

```
test('onPress gets called with the right thing', () => {
  const onPress = jest.fn();
  simulatePresses(onPress);
  expect(onPress).toBeCalledWith(
    expect.objectContaining({
      x: expect.any(Number),
      y: expect.any(Number),
    }),
  );
});

```

### expect.stringContaining(string)

- 'expect.stringContaining(string)'은 인자로 받은 값이 예상되는 문자에 정확하게 포함되어 있다면 일치하게 한다.

### expect.stringMatching(string | regexp)

- 'expect.stringMatching(string | regexp)'은 인자로 받은 값이 예상되는 정규식 패턴 또는 문자에 정확하게 포함되어 있다면 일치하게 한다.
- 리터럴 값과 대체해서 사용할 수 있다.

  - 'toEqual' 또는 'toBeCalledWith'
  - to match an element in arrayContaining
  - to match a property in objectContaining or toMatchObject

- 아래 예제는 'expect.arrayContaining' 내부에 'expect.stringMatching'을 사용하여 여러 비대칭 matcher를 중첩할 수 있는 방법을 보여줍니다.

```
describe('stringMatching in arrayContaining', () => {
  const expected = [
    expect.stringMatching(/^Alic/),
    expect.stringMatching(/^[BR]ob/),
  ];
  it('matches even if received contains additional elements', () => {
    expect(['Alicia', 'Roberto', 'Evelina']).toEqual(
      expect.arrayContaining(expected),
    );
  });
  it('does not match if received does not contain expected elements', () => {
    expect(['Roberto', 'Evelina']).not.toEqual(
      expect.arrayContaining(expected),
    );
  });
});
```

### expect.addSnapshotSerializer(serializer)

- 'expect.addSnapshotSerializer(serializer)'를 호출하여 'application-specific 데이터 구조'의 형식을 지정하는 모듈을 추가할 수 있습니다.
- 개별 테스트 파일의 경우 추가된 모듈은 내장 JS types 및 React Element에 대한 기본 스냅샷 serializers 앞에 오는 'snapshotSerializers' 구성의 어떤 모듈보다 우선시 됩니다.
  마지막으로 추가된 모듈이 테스트된 첫번째 모듈입니다.

```
import serializer from 'my-serializer-module';
expect.addSnapshotSerializer(serializer);

// affects expect(value).toMatchSnapshot() assertions in the test file
```

- snapshotSerializers 구성에 추가하는 대신 개별 테스트 파일에 스냅샷 serializer를 추가하는 경우
- 종속성을 암시적 대신 명시적으로 만듭니다.
- 'create-react-app'에서 배출할 수 있는 구성 제한을 피할 수 있습니다.
  <a href="https://jestjs.io/docs/configuration#snapshotserializers-arraystring/" target="_blank">추가 정보는 'configuring Jest'를 보세요.</a>
