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

