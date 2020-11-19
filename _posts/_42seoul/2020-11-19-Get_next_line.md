---
layout: post
title: "Get_next_line"
subtitle : "Get_next_line 서브젝트 풀이"
ㅇㅁㅅㄷ : 2020-11-19 23:11:42
background: '/img/posts/01.jpg'
---
# GNL

* **header**
```c

#ifndef GET_NEXT_LINE_H
# define GET_NEXT_LINE_H

# include <unistd.h>
# include <stdlib.h>

# ifndef BUFFER_SIZE
#  define BUFFER_SIZE 5000
# endif

# ifndef OPEN_MAX
#  define OPEN_MAX 32
# endif

int		get_next_line(int fd, char **line);
int		ft_strlen(char *s);
char	*ft_strchr(char *s, int c);
char	*ft_strdup(char *s1);
char	*ft_strjoin(char *s1, char *s2);

#endif
```

* **Description**
	* `BUFFER_SIZE` : read 할 때 한번에 읽어 올 수 있는 버퍼의 최대 사이즈
	* `OPEN_MAX` : 파일 오픈 제한 수를 정의하는 매크로
----
* **Fucntion**
```c
#include "get_next_line.h"

static int			get_line(char **temp, char **line, char *newline)
{
	char			*tmp;

	*newline = '\0';
	*line = ft_strdup(*temp);
	if (!(*(newline + 1)))
	{
		free(*temp);
		*temp = NULL;
	}
	else
	{
		tmp = ft_strdup(newline + 1);
		free(*temp);
		*temp = tmp;
	}
	return (1);
}

static int			check_temp(char **temp, char **line)
{
	char			*newline;

	if (*temp && (newline = ft_strchr(*temp, '\n')) != NULL)
		return (get_line(temp, line, newline));
	else if (*temp)
	{
		*line = *temp;
		*temp = NULL;
	}
	else
		*line = ft_strdup("\0");
	return (0);
}

int					get_next_line(int fd, char **line)
{
	static char		*temp[OPEN_MAX];
	char			buf[BUFFER_SIZE + 1];
	char			*newline;
	int				byte_count;

	newline = NULL;
	if (fd < 0 || !line || BUFFER_SIZE <= 0)
		return (-1);
	while ((byte_count = read(fd, buf, BUFFER_SIZE)) > 0)
	{
		buf[byte_count] = '\0';
		temp[fd] = ft_strjoin(temp[fd], buf);
		if ((newline = ft_strchr(temp[fd], '\n')) != NULL)
			break ;
	}
	if (byte_count < 0)
		return (-1);
	return (check_temp(&temp[fd], line));
}
```

 * **Description**			

	 * **함수 작동 기본 원리**
		 1. `main` 함수에서 `open`된 파일이 `fd`로 넘어 올 때 해당 파일을 읽을 수 있는 파일인지 혹은 읽을 수 있는 환경인지 검증한다.
		 2.  해당 파일을 `BUFFER_SIZE` 만큼 읽어 `buf`변수에 저장한 뒤, 읽어온 바이트 수를 `byte_count` 변수에 저장한다. 
		 3. 해당 파일에서 `'\n'` 개행 문자가 발견 될 때 까지 **2번**의 작업을 반복하되, 개행문자가 발견 되지 않고 `EOF`까지 도달하지 않을 때, `read` 함수를 통해 `buf`변수에 담긴 문자열을 정적 변수 `temp[fd]`에 `strjoin` 함수로 이어 붙인다.
		4. `EOF`에 도달하거나 `(4-1)` `EOF`에 도달하기 전`(4-2)`  *(`while ((byte_count = read(fd, buf, BUFFER_SIZE)) > 0)` 구문이 동작 하는 동안 즉 `byte_count`가 `0`이 아닐때)* **개행문자**를 발견 하면 해당 반복문을 `break`하고 `check_temp` 함수를 호출.
			
			`4-1` : `EOF`에 도달했음에도 개행문자를 찾지 못한 경우는 파일 전체에 개행문자가 1개도 없거나, 이미 **GNL**함수가 여러번 호출 되어 개행 문자가 없는 EOF 직전의 문자열이 `temp`변수에 담겨 있는 경우다.
			`4-2` : `EOF`에 도달하기 전에 개행 문자를 탐색한 경우 즉시 반복문을 중지하고 `line`에 옮기는 방식으로 코딩 하는 것이 파일 전체를 한번에 읽고 개행 문자 단위로 나눠 반환 하는 것보다 빠르다. 
		5. `check_temp` 함수에서 `temp`에 문자열이 들어있는지, 개행문자가 있는지 확인하고, 문자열이 존재하고 개행문자도 존재하는 경우 `get_line`함수를 호출한다.
		6. 위의 함수에서 `temp`에 문자열은 존재하지만 개행문자가 없는 경우 해당 문자열을 `line` 에 해당 문자열의 주소를 넘겨 준 뒤, `temp`변수가 가리키고 있던 메모리 주소를 초기화한다.
		7. **5번** 의 경우 처럼 개행문자가 발견되면 개행문자 전의 문자를 `line`변수에 옮겨 담고, 옮겨 담은 문자열을 `temp` 변수에서 옮겨 담은 문자열을 삭제한다.
