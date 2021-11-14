---
layout: post
title: "[React] useCallback과 React.memo"
subtitle: "React, useCallback, memo"
date: 2021-11-14 22:20:00
author: "dongjune"
header-img: "img/in_post/4.jpg"
catalog: true
tags:
  - React
---
# 🛫 Intro
이번 포스팅에서는 리액트의 최적화 기법 중 `useCallback`과 `memo`에 대해서 알아보도록 하겠습니다.  
리액트 컴포넌트가 리랜더링 될 때마다 그 안에 선언 된 함수들은 매번 재생성 되게 됩니다. 이때 `useCallback`을 사용하면 함수의 참조 값을 유지해줄 수 있습니다!  
코드를 살펴보며 제대로 이해하기 전에 우선 리액트가 언제 리랜더링이 일어나는지 확실히 알고 갈 필요가 있습니다! 

# React는 언제 리랜더링 될까?
리액트는 다음의 5가지 경우에서 리랜더링이 일어납니다.
1. state가 변경될 때
2. 새로운 props가 들어올 때
3. 부모 컴포넌트가 리랜더링 될 때
4. shouldComponentUpdate에서 true가 반환될 때
5. forceUpdate가 실행될 때

> 1, 2번의 경우 리액트는 기본적으로 얕은 비교를 통해 값을 비교합니다.  
state 값에 push나 pop 등의 원본을 변경하는 메소드를 사용하지 말아야 하는 이유는 리액트는 얕은 복사를 통해 값의 변경을 판단하기 때문입니다. 원본을 변경한다 해도 참조 값은 그대로이기 때문에 리랜더링은 일어나지 않게 됩니다.  

# 불필요한 리랜더링
부모 컴포넌트의 변경으로 인해 자식 컴포넌트들도 전부 랜더링 되는 경우를 생각해봅시다. 이때 자식 컴포넌트는 아무런 변경이 없어도 리액트는 리랜더링 되는 컴포넌트의 자식 컴포넌트들까지 전부 리랜더링 시킵니다.  
아래의 코드는 그 예시입니다. `Parent` 컴포넌트의 `count` 상태가 변경 되어 리랜더링이 일어나면 하위 컴포넌트인 Child 컴포넌트들도 모두 리랜더링이 일어나게 됩니다.
```jsx
import { useState } from "react";

export default function Parent() {
  const [player, setPlayer] = useState("Messi");
  const handleOnClick = (e) => {
    setPlayer(e.target.textContent);
  };
  return (
    <div>
      <h1>{player} is the best player!</h1>
      <Child onClick={handleOnClick} player={"Messi"} />
      <Child onClick={handleOnClick} player={"Ronaldo"} />
      <Child onClick={handleOnClick} player={"Son"} />
    </div>
  );
}

const Child = ({ onClick, player }) => {
  return <button onClick={onClick}>{player}</button>;
};
```
profiler로 확인해보면 Parent와 3개의 Child 컴포넌트 모두 리랜더링 되는 것을 볼 수 있습니다. 렌더링 시간은 총 3.4ms 만큼 걸리네요.
![1](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/2ae4d667-c27e-4718-8ccd-276f65f13481/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2021-11-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.19.59.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211114T142012Z&X-Amz-Expires=86400&X-Amz-Signature=0e0ad11ccb656e3c07aa3edb59ede4976fc35773109496f47f8db27e28732d7a&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202021-11-14%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%252011.19.59.png%22)

# React.memo로 컴포넌트를 메모이제이션하자!
`React.memo`를 사용하여 자식 컴포넌트들의 불필요한 리랜더링을 막을 수 있습니다. `React.memo`로 감싸진 컴포넌트는 들어오는 `props`의 변경이 없으면 리랜더링이 일어나지 않게 됩니다. 이때 주의할 점은 `React.memo`는 `props`에만 영향을 준다는 사실입니다. `useContext`나 `useState`에서 상태 값의 변경이 일어나면 `React.memo`로 메모이제이션 한 컴포넌트도 여전히 리랜더링이 일어나게 됩니다.  
또 한가지 주의할 점은 `React.memo`를 사용하는 컴포넌트는 동일한 `props`일때 동일한 랜더링 결과를 리턴해야 합니다.  
이제 위의 코드에서 `Child` 컴포넌트를 `React.memo`로 감싸보겠습니다.  
```jsx
const Child = React.memo(() => {
  return <div>Child</div>;
});
```
그 다음 profiler를 사용하여 랜더링을 측정해보겠습니다.  
`Parent` 컴포넌트 하나만 랜더링 되는것이 보이시나요? 랜더링 시간도 1ms로 전보다 무려 3배이상 빨라졌습니다!
![2](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/ce79895a-ea8a-49d7-a3e4-10298bd169b9/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2021-11-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.26.18.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211114T142627Z&X-Amz-Expires=86400&X-Amz-Signature=533819acdd5d354dd3cbe8937c0541d5474db7089262a29145a0d32b7bc96bbc&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202021-11-14%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%252011.26.18.png%22)

# 함수를 props로 넘겨주는 경우
그렇다면 `React.memo`로 메모이제이션 된 하위 컴포넌트들에게 함수를 `props`로 넘겨주는 경우는 어떨까요? 아래의 코드를 살펴보겠습니다.
```jsx
import React, { useState } from "react";

export default function Parent() {
  const [isClicked, setIsClicked] = useState(false);
  const handleOnClick = () => {
    setIsClicked(true);
  };
  return (
    <div>
      <h1>{isClicked ? "true" : "false"}</h1>
      <Child onClick={handleOnClick} />
      <Child onClick={handleOnClick} />
      <Child onClick={handleOnClick} />
    </div>
  );
}

const Child = React.memo(({ onClick }) => {
  return <button onClick={onClick}>Click me!</button>;
});

```
`Child`를 메모이제이션 했으니 `Child`들은 리랜더링이 일어나지 않을까요?
profiler를 확인해보겠습니다.
![3](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/2157c70b-6094-44fd-8f2c-e01e6754649c/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2021-11-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.51.23.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211114T145132Z&X-Amz-Expires=86400&X-Amz-Signature=3df329d3d9615ebd42c9c7361f843a979b8dbafaadb973ef661eb66c9856ae48&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202021-11-14%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%252011.51.23.png%22)
여전히 `Child` 컴포넌트들의 리랜더링이 일어나고 있습니다. 어떻게 된 걸까요?  
그것은 `props`로 넘겨주는 `handleOnClick` 함수가 Parent 컴포넌트의 리랜더링 마다 변경되기 때문입니다!  
기본적으로 컴포넌트의 리랜더링이 일어나면 그 안에서 선언된 함수들은 모두 재생성 되면서 참조 값이 변하게 됩니다. 리액트는 참조값을 통해 `props`를 비교하기 때문에 `Child`의 `props`가 변경됐다고 보고 `Child` 컴포넌트들을 리랜더링 시키는 것입니다.

# useCallback으로 함수의 참조값을 유지하자
위의 상황을 `useCallback`으로 해결할 수 있습니다.
사용법은 `useEffect`와 비슷합니다. 첫번째 인자로 메모이제이션을 수행할 `callback` 함수를 넣어주고 두번째 인자로 의존값을 넣어줍니다.  
```jsx
const handleOnClick = useCallback(() => {
    setIsClicked(true);
}, []);
```
위와 같은 경우 의존값을 빈배열로 설정했기 때문에 `handleOnClick` 함수는 `Parent` 컴포넌트가 처음 랜더링 될 때의 참조값을 유지하게 됩니다.  
이제 `Profiler`로 확인해보겠습니다.
![4](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/8942973c-4362-44c4-8ec2-532993b75553/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2021-11-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.55.54.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211114%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211114T145610Z&X-Amz-Expires=86400&X-Amz-Signature=a2528ec225a97fe7cebccd0a14d8038ccf30bffbf81e312782ddf61dd0a55f56&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202021-11-14%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%252011.55.54.png%22)  
보시는것과 같이 `Child` 컴포넌트들의 리랜더링이 일어나지 않게 됩니다. 랜더링 시간도 1.3ms 빨라졌네요.

# 🚀 Conclusion
이번 포스팅에서 useCallback과 React.memo에 대해서 알아봤습니다.  
사실 해당 최적화 기법들을 무작정 도입하는 것은 좋은 방법이 아닙니다. useCallback과 memo는 추가적인 비교 연산들을 수행하기 때문에 해당 hook을 사용해도 성능 차이가 별로 나지 않거나 최악의 경우 성능이 더 안좋아질 수도 있습니다.  
따라서 이러한 최적화를 도입할 때는 react devtool의 profiler를 사용하여 성능 검사와 함께 수행하는 것이 좋습니다.
# Reference