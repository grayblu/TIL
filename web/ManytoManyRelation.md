# M:N 관계를 활용한 커뮤니티 기능 구현

### 1. like

User : Board = M : N

### 2. Follow

User : User = M : N

새로운 USER 모델을 만들어 쓸 수 있도록 Django에서 제공한다. (AbstractUser)

※ 참고 문서

<https://docs.djangoproject.com/ko/2.2/topics/auth/customizing/#substituting-a-custom-user-model>



기존 회원 가입에서 사용했던 UserCreationFom() 재정의

<https://docs.djangoproject.com/ko/2.2/topics/auth/customizing/#custom-users-and-the-built-in-auth-forms>

```python
class CustomUserCreationForm(UserCreationForm):
     class Meta(UserCreationForm.Meta):
        model = get_user_model()
        fields = UserCreationForm.Meta.fields
```



※ 프로젝트를 다른 pc에서 작업 시 기존 환경을 그대로 반영시키기 위해 아래와 같은 명령어를 통해 .txt 파일을 만든다.

```bash
$ pip freeze > requirements.txt
```

