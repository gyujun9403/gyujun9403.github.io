---
layout: post
title: "MiniRT 동작 및 추가 개념 정리"
description: "42서울 MiniRT과제"
date: 2022-04-09
tags: [42Seoul, Raytracer, miniRT]
comments: true
share: true
---

이전 과제에서 다루었던  view 및 light(phong)개념은 생략

[이전 글](/_posts/2022-03-09-MiniRT_init.md)

## 1. 전체적인 동작 방식
![total flow](/images/42seoul/miniRT/total_flow.jpeg)
## 2. 구체적인 동작방식(자랑)
### 2.1 object 구조체 전달 방식
```c
typedef struct s_object	// 부모 클래스 같은 역할.
{
	int				type;
	t_ratio			ratio_reflect;
	struct s_object	*next;
}	t_object;

typedef struct s_sphere
{
	int				type;
	t_ratio			ratio_reflect;
	struct s_object	*next;
	t_point			center;
	double			r;
}	t_sphere;

typedef struct s_plane
{
	int				type;
	t_ratio			ratio_reflect;
	struct s_object	*next;
	t_point			point;
	t_vec			normal;
	double			r;
}	t_plane;

typedef struct s_cylinder
{
	int				type;
	t_ratio			ratio_reflect;
	struct s_object	*next;
	t_point			point;
	t_vec			normal;
	double			r;
	double			height;
}	t_cylinder;

typedef struct s_cone
{
	int				type;
	t_ratio			ratio_reflect;
	struct s_object	*next;
	t_point			point;
	t_point			c;
	t_vec			normal;
	double			r;
	double			height;
	double			theta;
}	t_cone;

...

t_hit	hit_object(t_ray ray, t_object *objects, int is_shadow)
{
	t_hit		min_ret;
	t_hit		tmp;
	t_object	*iter;

	iter = objects;
	min_ret.t = INF;
	min_ret.is_hit = FALSE;
	while (iter != NULL)
	{
		if (objects->type == TYPE_S)
			temp = hit_sphere(ray, (t_sphere *)objects);
		else if (objects->type == TYPE_P)
			temp = hit_plane(ray, (t_plane *)objects);
		else if (objects->type == TYPE_C)
			temp = hit_cylinder(ray, (t_cylinder *)objects);
		else
			temp = hit_cone(ray, (t_cone *)objects);
		if (tmp.is_hit == TRUE)
		{
			if (is_shadow == TRUE && (tmp.t >= EPSILON && tmp.t < min_ret.t))
				min_ret = tmp;
			else if ((tmp.t >= EPSILON && tmp.t < min_ret.t))
				min_ret = tmp;
		}
		iter = iter->next;
	}
	return (min_ret);
}
```
파싱한 각 오브젝트는 t_object형으로 cast된 리스트로 전달된다. t_object형은 각 object들이 앞부분에 가지는 공통 요소(type, ratio_reflect, next)들 구조체이다.

c++형태로 표현하면 t_object는 t_sphere, t_plane, t_ctlinder...들의 부모클래스처럼 사용 가능하다.

따라서 object를 순회할때 t_object형의 리스트로 순회된다. 그리고 각 오브젝트의 반별 함수에 type에 따라 적합한 object형으로 cast되어 전달된다.

### 2.2 화면 회전
- 정면으로 상하 θ도 회전 = z축(vertical) θ도 회전, y축(camera_direction) θ도 회전, x(horizon) 회전 안함.
- 정면에서 좌우 θ도 회전 = z축(vertical) 회전 안함, y축(camera_direction) θ도 회전, x(horizon) θ도 안함.

#### 2.2.1 상하 θ도 회전

(vx, vy, vz) = 변환 전 vertical 벡터

(hx, hy, hz) = 변환 전 horizon 벡터

(dx, dy, dz) = 변환 전 camera_direction 벡터

(a, b, c) = 변환 후 camera_direction

- (vx, vy, vz) · (a, b, c) = |1||1|cos(90' + θ)
- (hx, hy, hz) · (a, b, c) = |1||1|cos(90')
- (dx, dy, dz) · (a, b, c) = |1||1|cos(θ) 

![](/images/42seoul/miniRT/updown.png)

위의 식을 행렬식으로 표현하면

#### 2.2.2 좌우 θ도 회전

(vx, vy, vz) = 변환 전 vertical 벡터

(hx, hy, hz) = 변환 전 horizon 벡터

(dx, dy, dz) = 변환 전 camera_direction 벡터

(a, b, c) = 변환 후 camera_direction

- (vx, vy, vz) · (a, b, c) = |1||1|cos(90')
- (hx, hy, hz) · (a, b, c) = |1||1|cos(90' - θ)
- (dx, dy, dz) · (a, b, c) = |1||1|cos(θ)

![](/images/42seoul/miniRT/rightleft.png)

#### 2.2.3 종합

3*3역행렬을 구하는 함수를 만들고, 위에서 구한 식들을 곱하여, camera의 direction방향을 재설정하였다. 이후 다시 처음 픽셀부터 하나씩 계산하여 회전된 각도의 화면을 출력한다.