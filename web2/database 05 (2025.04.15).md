## Database 05 (2025.04.15)

### Many to Many Relationships

#### 1. Many to Many Relationships

- N:M or M:N
- 한 테이블의 0개 이상의 레코드가 다른 테이블의 0개 이상의 레코드와 관련된 경우 양쪽 모두에서 N:1 관계를 가짐

#### 2. ManyToManyField()

- M:N 관계 설정 모델 필드

#### 3. ‘through’ argument

- 중개 테이블에 ‘추가 데이터’를 사용해 M:N 관계를 형성하려는 경우에 사용

```python
# hospitals/models.py
from django.db import models

class Doctor(models.Model):
    name = models.TextField()

    def __str__(self):
        return f'{self.pk}번 의사 {self.name}'

class Patient(models.Model):
    doctors = models.ManyToManyField(Doctor, through='Reservation')
    name = models.TextField()

    def __str__(self):
        return f'{self.pk}번 환자 {self.name}'

class Reservation(models.Model):
    doctor = models.ForeignKey(Doctor, on_delete=models.CASCADE)
    patient = models.ForeignKey(Patient, on_delete=models.CASCADE)
    symptom = models.TextField()
    reserved_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f'{self.doctor.pk}번 의사의 {self.patient.pk}번 환자'
```

#### 4. MS:N 관계 주요 사항

- M:N 관계로 맺어진 두 테이블에는 물리적인 변화가 없음
- ManyToManyField는 중개 테이블을 자동으로 생성
- ManyToManyField는 M:N 관계를 맺는 두 모델 어디에 위치해도 상관 없음
    - 대신 필드 작성 위치에 따라 참조와 역참조 방향을 주의할 것
- N:1은 완전한 종속의 관계였지만, M:N은 종속적인 관계가 아니며 ‘의사에게 진찰 받는 환자 & 환자를 진찰하는 의사’이렇게 2가지 형태 모두 표현 가능

#### 5. ManyToManyField(to, **options)

- M:N 관계 설정 시 사용하는 모델 필드

#### 6. ManyToManyField 특징

- 양방향 관계
    - 어느 모델에서든 관련 객체에 접근할 수 있음
- 중복 방지
    - 동일한 관계는 한 번만 저장됨

#### 7. ManyToManyField의 대표 인자 3가지

1. related_name
2. symmetrical
3. through

#### 8. ‘related_name’ arguments

- 역참조 시 사용하는 manager name을 변경

```python
class Patient(models.Model):
    doctors = models.ManyToManyField(Doctor, related_name="patients")
```

```python
# 변경 전
doctor.patient_set.all()

# 변경 후 (변경 후 이전 manager name은 사용 불가
doctor.patients.all()
```

#### 9. ‘symmetrical’ arguments

- 관계 설정 시 대칭 유무 설정
- ManyToManyField가 동일한 모델을 가리키는 정의에서만 사용
- 기본 값
    - True
    
    ```python
    # 예시
    class Person(models.Model):
        friends = models.ManyToManyField("self")
        # friends = models.ManyToManyField("self", symmetrical=False)
    ```
    

- True일 경우
    - source 모델의 인스턴스가 target 모델의 인스턴스를 참조하면 자동으로 target 모델 인스턴스도 source 모델 인스턴스를 자동으로 참조하도록 함 (대칭)
    - default 값임
    - 즉, 내가 당신의 친구라면 자동으로 당신도 내 친구가 됨
    - 모델의 종류
        - source 모델
            - 관계를 시작하는 모델
        - target 모델
            - 관계의 대상이 되는 모델
- False일 경우
    - True와 반대 (대칭되지 않음)

#### 10. ‘through’ arguments

- 사용하고자 하는 중개 모델을 지정
- 일반적으로 “추가 데이터를 M:N 관계와 연결하려는 경우”에 활용

```python
class Patient(models.Model):
    doctors = models.ManyToManyField(Doctor, related_name="patients")
    name = models.TextField()

class Reservation(models.Model):
    doctor = models.ForeignKey(Doctor, on_delete=models.CASCADE)
    patient = models.ForeignKey(Patient, on_delete=models.CASCADE)
    symptom = models.TextField()
    reserved_at = models.DateTimeField(auto_now_add=True)
```

#### 11. M:N에서의 대표 조작 methods

- add()
    - 관계 추가
    - “지정된 객체를 관련 객체 집합에 추가”
- remove()
    - 관계 제거
    - “관련 객체 집합에서 지정된 모델 객체를 제거”

---

### 좋아요 기능 구현

#### 1. Article(M) - User(N)

- 0개 이상의 게시글은 0명 이상의 회원과 관련
- 게시글은 회원으로부터 0개 이상의 좋아요를 받을 수 있고, 회원은 0개 이상의 게시글에 좋아요를 누를 수 있음

#### 2. 모델 관계 설정

- Article 클래스에 ManyToManyField 작성

```python
# articles/models.py
class Article(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    """
    M:N관계 설정 시 그냥 복수형으로 이름을 짓는 것 보다,
    지금 우리가 만드는 기능이 무엇인지 생각해보고 명시적인 매니저 이름을 설정하는 것이 좋다.
    
    게시글 1번에 좋아요를 누른 모든 유저 조회
    (Article -> User / 참조)

    `게시글1.users.all()` 표현을 다음과 같이 표현하는 것이 더 적합하다. 
    `게시글1.like_users.all()`
    """
    """
    [역참조 매니저 이름 충돌 상황]
    - Article : User (N:1)
        - article.user (참조)
        - user.article_set (역참조)
    - Article : User (N:M)
        - article.like_users (참조)
        - user.article_set  (역참조)
        - user.like_articles (새로운 역참조 이름으로 변경)
    - N:1 관계의 역참조 이름(_set)은 바꾸지 않는 것을 권장한다.
        - 개발 시 _set 이름만 봤을 때 "아 지금  N:1에서 역참조를 하는것이구나"를 명시적으로 알기 위함이다.
    """
    like_users = models.ManyToManyField(settings.AUTH_USER_MODEL, related_name='like_articles')
    title = models.CharField(max_length=10)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

#### 3. 역참조 매니저 충돌

- like_users에 related_name을 설정해주지 않으면, 역참조 매니저 충돌이 일어남
- like_users 필드 생성 시 자동으로 역참조 매니저 .article_set가 생성됨
- 그러나 이전 N:1(Article-User) 관계에서 이미 같은 이름의 매니저를 사용 중 - user.article_set.all() → 해당 유저가 작성한 모든 게시글 조회
- ‘user가 작성한 글(user.article_set)’과 ‘user가 좋아요를 누른 글(user.article_set)’을 구분할 수 없게 됨
- user와 관계된 ForeignKey 혹은 ManyToManyField 둘 중 하나에 related_name 작성 필요

#### 4. User - Article 간 사용 가능한 전체 related manager

- article.user
    - 게시글을 작성한 유저 - N:1
- user.article_set
    - 유저가 작성한 게시글(역참조) - N:1
- article.like_users
    - 게시글을 좋아요 한 유저 - M:N
- user.like_articles
    - 유저가 좋아요 한 게시글(역참조) - M:N

#### 5. 금일 최종 코드

- articles/urls.py

```python
from django.urls import path
from . import views

app_name = 'articles'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:pk>/', views.detail, name='detail'),
    path('create/', views.create, name='create'),
    path('<int:pk>/delete/', views.delete, name='delete'),
    path('<int:pk>/update/', views.update, name='update'),
    path('<int:article_pk>/comments/', views.comments_create, name='comments_create'),
    path('<int:article_pk>/comments/<int:comment_pk>/delete/', views.comments_delete, name='comments_delete'),
    path('<int:article_pk>/likes/', views.likes, name='likes'),
]
```

- articles/models.py

```python
from django.db import models
from django.conf import settings

class Article(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    """
    M:N관계 설정 시 그냥 복수형으로 이름을 짓는 것 보다,
    지금 우리가 만드는 기능이 무엇인지 생각해보고 명시적인 매니저 이름을 설정하는 것이 좋다.
    
    게시글 1번에 좋아요를 누른 모든 유저 조회
    (Article -> User / 참조)

    `게시글1.users.all()` 표현을 다음과 같이 표현하는 것이 더 적합하다. 
    `게시글1.like_users.all()`
    """
    """
    [역참조 매니저 이름 충돌 상황]
    - Article : User (N:1)
        - article.user (참조)
        - user.article_set (역참조)
    - Article : User (N:M)
        - article.like_users (참조)
        - user.article_set  (역참조)
        - user.like_articles (새로운 역참조 이름으로 변경)
    - N:1 관계의 역참조 이름(_set)은 바꾸지 않는 것을 권장한다.
        - 개발 시 _set 이름만 봤을 때 "아 지금  N:1에서 역참조를 하는것이구나"를 명시적으로 알기 위함이다.
    """
    like_users = models.ManyToManyField(settings.AUTH_USER_MODEL, related_name='like_articles')
    title = models.CharField(max_length=10)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

class Comment(models.Model):
    article = models.ForeignKey(Article, on_delete=models.CASCADE)
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    content = models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)from django.urls import path
```

- articles/views.py

```python
from django.contrib.auth import login as auth_login
from django.contrib.auth import logout as auth_logout
from django.contrib.auth import update_session_auth_hash, get_user_model
from django.contrib.auth.forms import AuthenticationForm, PasswordChangeForm
from django.shortcuts import redirect, render

from .forms import CustomUserChangeForm, CustomUserCreationForm

def login(request):
    if request.user.is_authenticated:
        return redirect('articles:index')

    if request.method == 'POST':
        form = AuthenticationForm(request, request.POST)
        if form.is_valid():
            auth_login(request, form.get_user())
            return redirect('articles:index')
    else:
        form = AuthenticationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/login.html', context)

def logout(request):
    auth_logout(request)
    return redirect('articles:index')

def signup(request):
    if request.user.is_authenticated:
        return redirect('articles:index')

    if request.method == 'POST':
        form = CustomUserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('articles:index')
    else:
        form = CustomUserCreationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/signup.html', context)

def delete(request):
    request.user.delete()
    return redirect('articles:index')

def update(request):
    if request.method == 'POST':
        form = CustomUserChangeForm(request.POST, instance=request.user)
        if form.is_valid():
            form.save()
            return redirect('articles:index')
    else:
        form = CustomUserChangeForm(instance=request.user)
    context = {
        'form': form,
    }
    return render(request, 'accounts/update.html', context)

def change_password(request, user_pk):
    if request.method == 'POST':
        form = PasswordChangeForm(request.user, request.POST)
        if form.is_valid():
            user = form.save()
            update_session_auth_hash(request, user)
            return redirect('articles:index')
    else:
        form = PasswordChangeForm(request.user)
    context = {
        'form': form,
    }
    return render(request, 'accounts/change_password.html', context)

def profile(request, username):
    # username으로 어떤 유저인지 조회
    # get_user_model().objects.get(username=username)
    User = get_user_model()
    person = User.objects.get(username=username)
    context = {
        'person': person,
    }
    return render(request, 'accounts/profile.html', context)

def follow(request, user_pk):
    # 팔로우 할 상대방 조회
    User= get_user_model()
    you = User.objects.get(pk=user_pk)
    me = request.user

    # 내가 나를 팔로우 하려는 것은 아닌지 먼저 확인
    # 나와 저 사람이 다른 사람일 경우 팔로우 로직 진행
    if me != you:
        # 좋아요를 추가하는 건지 / 취소하는 건지 구분
        # 상대방의 팔로워 목록에 내가 있는지 없는지를 확인
        if me in you.followers.all():
            you.followers.remove(me)
            # me.followings.remove(you)
        else:
            you.followers.add(me)
            # me.followings.add(you)
    return redirect('accounts:profile', you.username)
```

- articles/index.html

```html
{% comment %} articles/index.html {% endcomment %}
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <h1>Articles</h1>
  {% comment %} {% if user.is_authenticated %} {% endcomment %}
  {% if request.user.is_authenticated %}
    <h1>안녕하세요, {{ user.username }}님</h1>
    <a href="{% url "accounts:profile" user.username %}">내 프로필</a>
    <a href="{% url "articles:create" %}">CREATE</a>
    <form action="{% url "accounts:logout" %}" method="POST">
      {% csrf_token %}
      <input type="submit" value="로그아웃">
    </form>
    <form action="{% url "accounts:delete" %}" method="POST">
      {% csrf_token %}
      <input type="submit" value="회원탈퇴">
    </form>
    <a href="{% url "accounts:update" %}">회원정보 수정</a>
  {% else %}
    <a href="{% url "accounts:login" %}">로그인</a>
    <a href="{% url "accounts:signup" %}">회원가입</a>
  {% endif %}

  <hr>

  {% for article in articles %}
    <p>
      작성자 : 
      <a href="{% url "accounts:profile" article.user.username %}">{{ article.user }}</a>
    </p>
    {% comment %} <p>작성자 : {{ article.user.username }}</p> {% endcomment %}
    <p>글 번호: {{ article.pk }}</p>
    <a href="{% url "articles:detail" article.pk %}">
      <p>글 제목: {{ article.title }}</p>
    </a>
    <p>글 내용: {{ article.content }}</p>
    {% comment %} 좋아요 버튼 {% endcomment %}
    <form action="{% url "articles:likes" article.pk %}" method="POST">
      {% csrf_token %}
      {% if request.user in article.like_users.all %}
        <input type="submit" value="좋아요 취소">
      {% else %}
        <input type="submit" value="좋아요">
      {% endif %}
    </form>
    <hr>
  {% endfor %}
</body>
</html>
```

- accounts/profile.html

```html
{% comment %} accounts/profile.html {% endcomment %}
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <h1>{{ person.username }}의 프로필 페이지</h1>
  {% comment %} 팔로잉 / 팔로워 수 출력 {% endcomment %}
  <div>
    팔로잉 : {{ person.followings.all|length }} / 팔로워 : {{ person.followers.all|length }}
  </div>

  {% comment %} 팔로우 버튼 {% endcomment %}
  {% if request.user != person  %}
    <form action="{% url "accounts:follow" person.pk %}" method="POST">
      {% csrf_token %}
      {% if request.user in person.followers.all %}
        <input type="submit" value="언팔로우">
      {% else %}
        <input type="submit" value="팔로우">
      {% endif %}
    </form>
  {% endif %}

  <hr>
  <h2>{{ person.username }}가 작성한 게시글</h2>
  {% for article in person.article_set.all %}
    <p>{{ article.title }}</p>
  {% endfor %}
  <hr>
  <h2>{{ person.username }}가 작성한 댓글</h2>
  {% for comment in person.comment_set.all %}
    <p>{{ comment.content }}</p>
  {% endfor %}
  <hr>
  <h2>{{ person.username }}가 좋아요를 누른 게시글</h2>
  {% for article in person.like_articles.all %}
    <p>{{ article.title }}</p>
  {% endfor %}
</body>
</html>
```
