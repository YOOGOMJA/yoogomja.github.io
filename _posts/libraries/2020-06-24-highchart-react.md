---
layout: post
title: "React에서 Highchart 사용하기"
subtitle: "React에서 쓸 수 있는 강력한 차트 라이브러리"
author: "yoogomja"
header-style: text
tags:
    - react
    - highchart
---

몇 년 전, 회사에서 화면 작업에 차트를 그려야할 일이 있어서 찾아보다가 `highchart`라는 라이브러리를 찾게 되었다. 굉장히 많은 종류의 차트를 만들 수 있었고, 무엇보다 구현이 굉장히 쉬운 편이었다. 옵션들도 다양하지만 문서를 참고한다면 어렵지 않게 구현할 수 있는 차트였다. 

상업적인 용도로 사용한다면, 별도의 비용을 지불해야하지만 개인 사용자가 비 상업적인 용도로 사용한다면 무료로 제공하고있기 때문에 작은 프로젝트에서 빠르게 차트를 적용하는데에 문제없을 것으로 보여서 사이드 프로젝트를 진행하면서 사용하게 되었다. 

`typescript`로 개발되어있는 환경에서도 차트를 대응할 수 있고, 몇가지 스타일 예제를 제공해서 테마처럼 사용할 수 있도록 설정할 수도 있다. 차트 종류는 간단한 막대 그래프, 파이 차트부터 복잡한 히트맵까지 넓은 범위를 커버하고 있다. 이 예제들은 [여기](https://www.highcharts.com/demo?_ga=2.222711771.1722427230.1592983618-2136624288.1592983618)에서 확인할 수 있다.

기본적으로 문서에서 설치와 실행 예제만 보더라도 금방 사용할 수 있지만, `react`와 `typescript`를 사용하는 환경에서는 약간의 추가조작이 필요하다. 

# 1. 설치하기 

설치는 `yarn`이나 `npm`으로 설치하면 된다.

```sh
yarn add highcharts
yarn add highcharts-react-official

npm install highcharts --save
npm install highcharts-react-official --save
```

# 2. 사용하기

사용하는 것도 어렵지는 않다. 다만, 차트의 옵션이 변경되면 즉, 데이터가 변경되면 해당 항목을 변경해서 새로 주입해주어야 하므로 `state`를 사용해야 한다. 


```tsx
import react , {useState, useEffect} from 'react';
import * as Highcharts from 'highcharts';
import HighchartsReact from 'highhcarts-react-official';

const ExampleComponent = ()=>{
    const initialOptions: any = {
        title : { text : "example" },
        chart : { type : "pie" },
        series : [] // 데이터가 처음엔 비어았다.
    };  
    const [ options, setOptions ] = useState<any>(initialOptions);

    const asyncRequest = async ()=>{
        const result = await some_async_request(); 
        // 임의의 비동기 요청이 있다고 가정한다.
        // result의 data안에는 text라는 문자열과, value라는 값이 저장되어있다고 가정

        let tempSeries = [];
        result.data.forEach(item=>tempSeries.push({
            name : item.text,   // 요소의 이름
            y: item.value       // 값 
        }));

        // 옵션을 변경하면 자동으로 Highcharts가 갱신된다.
        setOptions({
            ...initialOptions,
            series : tempSeries
        });
    }

    useEffect(()=>{
        asyncRequest();
        return ()=>{
            setOptions(initialOptions);
        }
    },[]);

    return <div>
        <HighchartsReact 
            highcharts={ Highcharts } 
            options={ options }/>
    </div>;
}

```

위의 코드에서는 초기화용 옵션을 미리 선언해두고, 데이터를 조회한 후에는 `series`부분만 변경하도록 한다. `series`는 실제 데이터 값이 주로 포함되는 부분이다. `state`를 사용해 데이터를 변경하면 무리없이 비동기로도 데이터를 갱신해줄 수 있다. 

이 방법을 응용해, 데모 페이지에서 제공하는 다양한 테마 내용들을 미리 옵션형태로 만들어두고 불러와서 사용하면 모든 차트에서 동일한 테마 형태를 유지할 수있다. 

