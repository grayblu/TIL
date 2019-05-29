# Bootstrap4 복습

### 0. 들어가기 전에

Bootstrap을 통한 스타일 설정은 클래스 선택자로 한다

### 1. spacing

브라우저 내 표현되는 box의 요소 중 margin과 padding 수정하는 방법에 대해 학습했다.

```html
<div class="p-2 mt-2">
    box의 padding과 margin top은 2 
</div>
```



### 2. Grid

Bootstrap의 핵심 기능이라고 생각한다. Container 내에 12개의 열로 균등 분할이 가능하다



### 3. Animation

[animate.css](<https://daneden.github.io/animate.css/>)에서 간편하게 사용할 수 있으며, CDN을 지정하고 클래스 내에 원하는 기능을 추가하여 손쉽게 꾸밀 수 있다.



### 4. Flex

CSS에서 flex-direction은 4방향(위, 아래, 좌, 우)이며 그리드 정렬은 justfy-content와 align-items 속성을 이용한다.

```css
.container {
              flex-direction: row | row-reverse | column | column-reverse;
  			  /* 오른쪽 정렬 */
              justify-content: flex-end;
              /* 가운데 정렬 */
              justify-content: center;
    
    		  /* 상단 정렬 */
              align-items: flex-start;
              /* box의 아래 선을 기준으로 정렬(font size가 다를 경우 사용) */
              align-items: baseline;
}
```

해당 속성명은 Bootstrap에서도 거의 동일하게 사용되며 아래와 같이 지정한다.

```html
<div class="container">
    <div class="row justify-content-center">
        <div class="col-2">1</div>
        <div class="col-2">2</div>
        <div class="col-2">3</div>
    </div>
</div>
```



### 5. 끝

Bootstrap은 CSS에 대한 기초를 바탕으로 빠르고 간편하게 브라우저를 꾸밀 수 있는 도구이다. 하지만 브라우저의 디테일까지 신경쓰기 위해서는 CSS와 함께 쓸 수 있는 훈련이 필요하다. 그렇기에 다가올 프로젝트까지 CSS와 Bootstrap을 자율롭게 사용할 수 있도록 꾸준히 학습해야겠다.