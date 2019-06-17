# Auth(인증, 허가)

### 1. sign up

- CRUD의 CREATE에 해당
- User 모델을 만들 필요가 없다.
- UserCreationForm()을 통해 회원가입란을 생성한다.

```python
from django.contrib.auth.forms import UserCreationForm
```

### 2. login

- Session Create

- AuthenticationForm()을 통해 사용자 정보를 인증한다.
- `django.contrib.auth`에서 `login `함수를 통해 사용자 세션을 생성하고, 일정 시간동안 유지할 수 있도록 한다.

※ 세션: 일정 시간동안 같은 클라이언트로부터 들어오는 요청을 하나의 상태로 보고 유지하는 기술. 클라이언트도 세션ID를 가지고 있으며, 쿠키가 삭제될 경우 클라이언트 정보가 다르기 때문에 로그인과 같은 정보가 초기화된다.

```python
def login(request):
    if request.method == 'POST':
        form = AuthenticationForm(request, request.POST)
        if form.is_valid():
            auth_login(request, form.get_user()) # user session create
            return redirect('boards:index')
    else:
        form = AuthenticationForm()

    content = { 'form': form, }
    return render(request, 'accounts/login.html', content)

```

### 3. logout

- request를 통해 들어오는 사용자 정보를 포함한 Session을 삭제한다.
- `django.contrib.auth`에서 `logout `함수를 통해 사용자 세션을 삭제

※ 로직 조정

로그인 상태에서만 게시글 작성, 회원가입 후 세션 추가, 로그인 상태에서는 로그아웃 / 로그인하지 않았다면 회원가입과 로그인

### 4. 회원 탈퇴

- `request.user.delete()`

### 5. 회원 수정

- UserChangeForm()에 제공하는 수정 항목이 관리자 권한에서 사용하는 것들이 많으므로 커스터마이징이 필요함
- UserChangeForm을 상속받아 수정 항목을 재정의한다.

```python
class CustomUserChangeForm(UserChangeForm):
    class Meta:
        model = get_user_model()
        fields = ('email', 'first_name', 'last_name',)
```

- 수정 항목 선정 관련하여 공식 문서 내 User object를 확인한다. - [user object 정보 링크](https://docs.djangoproject.com/en/2.2/topics/auth/default/#using-the-django-authentication-system)

- 뷰 함수에서 CustomUserChangeFom 인스턴스 생성하고 유효성 검증과 함께 수정 사항을 저장한다.

```python
def update(request):
    if request.method == 'POST':
        form = CustomUserChangeForm(request.POST, instance=request.user)
        if form.is_valid():
            form.save()
            return redirect('boards:index')
    else:
        form = CustomUserChangeForm(instance=request.user)
    context = {'form': form, }
    return render(request, 'accounts/update.html', context)

```

### 6. 비밀번호 변경

- PasswordChangeForm() 이용
- 현재 사용자의 인증 세션 무효화를 막고 세션 유지를 위해 update_session_auth_hash()를 사용

```python
def change_password(request):
    if request.method == 'POST':
        form = PasswordChangeForm(request.user, request.POST)
        if form.is_valid():
            user = form.save()
            update_session_auth_hash(request, user)
            return redirect('boards:index')
    else:
        form = PasswordChangeForm(request.user)
    context ={ 'form': form,}
    return render(request, 'accounts/change_password.html', context)
```



### 7. Authorization

- 사용자 세션이 없는 상태에서 URL을 통한 게시글 작성, 수정 등의 페이지 접근 권한을 부여하기 위한 기능을 추가
- 접근 권한이 필요한 뷰 함수에 @login_required를 추가한다.
- 게시물 추가, 수정 url 접근 시 로그인 페이지로 리다이렉트하게 되고 이때의 url(`accounts/login/?next=/boards/create/`) 중 속성이 next에 해당되는 값(로그인 정보)을 받으면 요청했던 페이지로 접근 가능하게 함.

### 8. model에 user_id 추가

- `models.py`에서 사용자에 대한 컬럼을 추가한다. 사용자는 Django에서 제공하는 모델이며 아래와 같이 추가하면 된다.

```python
from django.db import models
from django.conf import settings

# Create your models here.
class Board(models.Model):
    ...
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
```

- 모델 수정 이후에 진행하는 migration 작업 진행 후, 게시글 추가에서 user_id가 not null 속성이 되기 위해 뷰 함수를 수정한다.

```python
@login_required
def create(request):
    if request.method == 'POST':
			...
            # ModelForm을 적용 이후
            # board 를 바로 db 에 저장하지 않고 user 정보를 받은 후에 저장
            board = form.save(commit=False)
            board.user = request.user
            board.save()
            return redirect('boards:detail', board.pk)
    # GET: 기본 form 인스턴스를 생성
    else:
        form = BoardForm()
    # GET: 기본 form 의 모습
    # POST: 요청에서 검증에 실패한 form 이며 오류메시지를 포함한 상태임
    context = {'form':form}
    return render(request, 'boards/form.html', context)
```

