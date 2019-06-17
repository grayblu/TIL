※ 모델을 만드는 순서

1. models.py 작성

2. makemigrations

3. migrate

model.py에서 클래스를 정의하여 필드를 구성한다. 간단한 예로 제목과 내용을 포함한 클래스를 정의했다.

```python
from django.db import models

# Create your models here.
class Board(models.Model):
    title = models.CharField(max_length=20)
    content = models.CharField(max_length=200)

```

django 모델을 정의할 때 타입은 아래의 링크를 참조할 것

[django model type](<https://docs.djangoproject.com/es/1.10/ref/models/fields/#field-types>)

```bash
$ python manage.py makemigrations board

$ python manage.py migrate

$ pyhon manage.py shell
```

makemigrations 명령을 통해 migiration 디렉토리 내에 0001_initial.py를 만들고 ORM이 가능한 형태로 변환한 클래스를 자동으로 정의한다.

이후 migrate 명령을 통해 ORM이 이뤄지도록 한다.(DB 접근, 제어)

명령어를 통해 DB를 조회하며 ORM이 제어한다.

DB ----- ORM ----- Python

models.py에서 Board라는 클래스가 있고, 필드를 title, content로 지정했다.

### ORM

#### Create

첫 번째 

```bash
>>> Board.objects.all()
<QuerySet []>
>>> board = Board()
>>> board.title = 'first'
>>> board.content = 'django!'
>>> board.save()
```



두 번째

```bash
>>> board = Board(title='second', content='django!!')
>>> board.save()
```

세 번째

```bash
>>> Board.objects.create(title='third', content='django!!!')
<Board: Board object (3)>
```

※ 해당 경우는 작성 완료와 함께 저장이 된다. 유효성 검증이 어렵다.



#### Read

get 메서드를 통해 id 값에 해당하는 컬럼을 가져온다.(id 또는 pk)

```bash
>>> board = Board.objects.get(id=1)
>>> boards
<Board: 1글 - first: django!>
```

필터 기능을 이용하여 다양한 방법으로 조회도 할 수 있다.

```bash
>>> boards = Board.objects.filter(title='first')
>>> boards
<QuerySet [<Board: 1글 - first: django!>]>

```

filter는 장고에서 하나 이상의 데이터가 존재한다고 가정하기 때문에 쿼리셋으로 리턴한다.

```bash
>>> boards = Board.objects.filter(title='first').first()
>>> boards
<Board: 1글 - first: django!>
```

하나의 컬럼을 가져온다

필터를 통해 값의 일부를 가지고 조회가능하다.

```bash
>>> boards = Board.objects.filter(title__contains='fi')
>>> boards
<QuerySet [<Board: 1글 - first: django!>]>
>>> boards = Board.objects.filter(content__endswith='!')
>>> boards
<QuerySet [<Board: 1글 - first: django!>, <Board: 2글 - second: django!!>, <Board:
3글 - third: django!!!>, <Board: 4글 - fourth: django!!!!>]>

```

오름차순, 내림차순 정렬도 가능하며, 내림차순은 데이터 앞에 '-'를 붙인다.

```bash
>>> boards = Board.objects.order_by('title')
>>> boards
<QuerySet [<Board: 1글 - first: django!>, <Board: 4글 - fourth: django!!!!>, <Board
: 2글 - second: django!!>, <Board: 3글 - third: django!!!>]>


>>> boards = Board.objects.order_by('-title')
>>> boards
<QuerySet [<Board: 3글 - third: django!!!>, <Board: 2글 - second: django!!>, <Board
: 4글 - fourth: django!!!!>, <Board: 1글 - first: django!>]>

```

슬라이싱 기능

```bash
>>> boards[1]
<Board: 2글 - second: django!!>
>>> boards[1:3]
<QuerySet [<Board: 2글 - second: django!!>, <Board: 4글 - fourth: django!!!!>]>

```

#### Update

수정

```bash
>>> board.title = 'byebye'
>>> board.save()
>>> board
<Board: 1글 - byebye: django!>
```

save()는 board 객체에 id가 없을때는 create하고, 있으면 update

삭제 

```bash
>>> board.delete()
(1, {'boards.Board': 1})
```



### Web 에서 crud를 통한 페이지 구현

#### ※ 기본적으로 DB 조작이므로 POST방식을 이용한다.

- 데이터 관련 정보는 url에 노출되면 안된다.

- 그리고 최소한의 신원확인이 필요하므로 csrf token을 통해 검증된 요청을 받아야 한다.(form tag 안에 **{% csrf_token %} 추가** 필요함)

- POST 요청은 렌더링이 아니므로 요청 처리 후 결과를 보여주기 위해 다른 페이지로 보내야 한다.(Redirect)

#### Create

1. 글을 작성할 페이지를 띄워줌

2. 입력 데이터를 바탕으로 Table 내 레코드 추가 

- create()에서 객체 인스턴스 생성
- GET을 통해 전달받은 인자를 ORM을 통해 컬럼에 추가

※ dir(인스턴스명) 을 통해 사용 가능한 모든 메서드를 확인할 수 있다.



### 게시글에 댓글 남기기

Board : Comment = 1: N,	Foreignkey 관계 사용

Comment 모델 정의

```python
class Comment(models.Model):
    # on_delete=models.CASCADE => 모델 삭제 시 관련 테이블의 컴럼도 삭제 
    board = models.ForeignKey(Board, on_delete=models.CASCADE)
    content = models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

board는 테이블 생성 시 컬럼명이 board_id로 자동 변경된다.

1에서 N을 참조할 때 - .comment_set

**N에서 1을 참조할 때 - .board**

#### 댓글 생성 기능 구현

특정 게시글에 댓글 입력란과 출력될 수 있도록 template 수정한다.

form 의 정보를 받아 Comment 테이블에 내용을 추가해야 하므로 method는 POST 방식!

특정 게시글에 대한 댓글 기능이므로 POST로 전달 시 board_pk가 넘어와야 함

그렇다면 view 함수를 작성하는 과정에서 POST 방식 여부에 따라 return 페이지를 달리 하며 여기서 Tip으로 **pass**를 통해 보다 효율적으로 작성 가능

```python
def comment_create(request, board_pk):
    if request.method == 'POST':
        pass
    else:
        return redirect('boards:detail', board.pk)
```

post 방식에 대한 기능 구현 과정이 다소 길고 뷰함수가 제대로 정의되지 않으면 서버 실행이 불가하기 때문에 위와 같은 방식으로 페이지를 확인하면서 기능을 구현할 수 있다.

최종적으로 댓글 생성에 대한 기능을 구현한  comment_create 함수는 아래와 같다.

```python
def comments_create(request, board_pk):
    # 댓글을 달 게시물
    board = Board.objects.get(pk=board_pk)
    if request.method == 'POST':
        # form 에서 넘어온 댓글 정보
        content = request.POST.get('content')
        # 댓글 생성 및 저장
        comment = Comment(board=board, content=content)
        # comment = Comment(board_id=board.pk, content=content)
        comment.save()
        return redirect('boards:detail', board.pk)
        # return redirect('boards:detail', comment.board_id)
    else:
        return redirect('boards:detail', board.pk)
```

이 기능만으로는 DB에 댓글이 추가만 되고 출력되지 않으므로, detail 함수에서 댓글을 불러 출력할 수 있도록 한다. 

#### 댓글 삭제



### 구현 중에 겪은 예외 상황

```
django.urls.exceptions.NoReverseMatch:
```

url 구성이 잘못된 경우 => url 입력하는 코드를 살핀다. 또는 template 에서 받는 인자 값을 확인 것.(variable routing인지 object인지)

### 이미지 추가, 조회

#### 기본 설정 순서

**모듈 추가**

```bash
$ pip install pillow
$ pip install pilkit
$ pip install django-imagekit
```



**models.py에서 image 추가**

```python
from imagekit.models import ProcessedImageField
from imagekit.processors import Thumbnail
class Board(models.Model):
    ...
    
    image = ProcessedImageField(
            upload_to='boards/images',   # 저장 위치 (media 이후의 경로)
            processors=[Thumbnail(200,300)],    # 처리할 작업 목록
            format='JPEG',  # 저장 포맷
            options={'quality': 90},    # 추가 옵션
        )
    # image = models.ImageField(blank=True)
```

초기에 ImageField를 이용해 타입을 설정했으나, django-imagekit 모듈을 이용하여 재정의 했다.

모델을 수정했으니 이후 bash에서 `makemigrations`와 `migrate`명령을 통해 ORM 적용되도록 세팅한다.







`<form>`태그에서 이미지를 업로드 할 경우 반드시 `enctype`특성을 설정한다.

```html
<form action="{% url 'boards:update' board.pk %}" method="POST" enctype="multipart/form-data">
</form>
```



### ※ favicon 추가

16x16, 32x32의 png 확장자 이미지 파일이며, 설정은 아래와 같다.

```html
<link rel="shotcut icon" type="image/png" href="{% static 'boards/images/favicon.png' %}">
```



### Django의 Form 기능을 이용하여 CRUD

local apps 디렉토리에 forms.py를 만들고, 

사용할 form에 대하여 클래스를 아래와 같이 정의한다.

```python
from django import forms

class BoardForm(forms.Form):
    title = forms.CharField(max_length=50)
    content = forms.CharField()
```

클래스 정의 후 , `views.py`에서 해당 클래스 import를 하고 뷰 함수에서 인스턴스 생성한다.

```python
form = BoardForm()
```

form을 GET 방식으로 전달할 경우 Dict 구조로 페이지에 전달하고 해당 페이지에서 

`{{ form }}`을 입력하면 Django 에서 라벨과 입력창이 담긴 폼을 출력한다.

조금 더 디테일하게 form에 대한 수정도 아래와 같이 할 수 있다.

```python
class BoardForm(forms.Form):
    title = forms.CharField(
        max_length=50,
        label='제목',
        widget=forms.TextInput(
            attrs={
                'class':'title',
                'placeholder': 'Enter the title'

            }
        )

    )
    content = forms.CharField(
        label='내용',
        widget=forms.Textarea(
            attrs={
                'class': 'content-type',
                'rows': 5,
                'cols': 50,
                'placeholder': 'Enter the content',
            }
        )

    )
```

CRUD에서 Create, Update가 Form을 사용하므로 이전 



```bash
In [4]: board.__dict__
Out[4]:
{'_state': <django.db.models.base.ModelState at 0x6b9f438>,
 'id': 5,
 'title': 'fifth_ddffff',
 'content': 'fifth testssssss',
 'created_at': datetime.datetime(2019, 6, 13, 1, 23, 3, 451804, tzinfo=<UTC>),
 'updated_at': datetime.datetime(2019, 6, 13, 2, 13, 53, 960804, tzinfo=<UTC>)}
```







#### ※ 파이썬 코드 실행 메커니즘을 확인할 때

```
from IPython import embed

'''
# 코드를 중지할 부분에 embed() 삽입
embed()
```



#### ※ 사용자에게 페이지 오류메시지를 표시하는 방법