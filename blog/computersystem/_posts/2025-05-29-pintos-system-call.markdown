---
layout: post
title:  "PintOS 2~3주차 argument passing & system call 구현"
description: >
 이 글에서는 PintOS 2~3주차 argument passing & system call 구현에 관련하여서 다루겠다.
date:   2025-05-29
image: /assets/img/blog/postimage/ComputerSystem.png
hide_last_modified: true
---

* toc  
{:toc .large-only}

저번주에 muliple-thread까지 구현하고 이번주 부터는 argument passing과 system call에 대해서 구현을 해야 한다.

## Argument Passing

Argument passing을 다루기 전에 먼저 알아두어야 하는 것들이 있다. 

- 커널은 User program이 실행되는 것을 허가하기 전에, 레지스터에 올라가 있는 초기 함수를 위한 인자를 반드시 넣어줘야 함, 이 인자들은 호출 규약과 동일한 방식으로 전달됨.

### 호출 규약

1. User level application은 `%rdi`, `%rsi`, `%rdx`, `%rcx`, `%r8`, `%r9` 시퀸스들을 전달하기 위해 정수 레지스터 사용

2. 호출자는 다음 인스트럭션의 주소(리턴 어드레스)를 스택에 푸시하고, 피호출자의 첫번째 인스트럭션으로 점프함. `CALL` 이라는 x86-64 인스트럭션 하나가 이 두가지를 모두 수행

3. 피호출자가 실행됨

4. 만약 피호출자가 리턴 값을 가지고 있다면, 리턴 값은 `%rax`에 저장됨

5. 피호출자는 x86-64 인스트럭션인 `RET` (리턴)를 사용해서, 스택에 받았던 리턴 어드레스를 pop하고 그 주소가 가리키는 곳으로 점프함으로써 리턴됨

#### 호출 규약 예시

세 개의 정수 인자를 받는 함수 `f()` 가 있다고 생각해보자

아래 도식은 위의 3번 항목에 있는 피호출자가 실행되는 시점에, 스택 프레임과 레지스터 상태가 어떤 식으로 되어있는지에 대한 예시를 보여줌. `f()`가 `f(1, 2, 3)` 으로 호출되었다고 가정합시다.

초기화된 스택의 주소는 임의의 숫자로 치면 다음과 같음

~~~txt
                             +----------------+
stack pointer --> 0x4747fe70 | return address |
                             +----------------+
RDI: 0x0000000000000001 | RSI: 0x0000000000000002 | RDX: 0x0000000000000003
~~~

### Argument Parsing

`/bin/ls -l foo bar` 와 같은 명령어 주어졌을 때, 인자들을 어떻게 다뤄야 하는지 생각해야 한다.

1. 명령을 단어들로 쪼개야 한다. `/bin/ls`, `l`, `foo`, `bar` 으로

2. 이 단어들을 스택의 맨 처음 부분에 놓는다. 순서는 상관이 없다. 왜냐면 포인터에 의해 첨조될 예정이기 때문

3. 각 문자열의 주소 + 경계조건을 위한 널포인털를 스택에 오른쪽 → 왼쪽 순서로 푸시. 이들은 argv의 원소가 됨. 널포인터 경계는 argv[argc]가 널포인터라는 사실을 보장해줌. c언어 표준의 요구사항에 맞추기 때문. 그리고 이 순서는 argv[0] 이 가장 낮은 가상 주소를 가진다는 사실을 보장. 또한 word 크기에 정렬된 접근이 정렬되지 않은 접근보다 빠르므로, 최고의 성능을 위해서는 스택에 첫 푸시가 발생하기 전에 스택포인터를 8의 배수로 반올림하여야 함.

4. `%rsi` 가 `argv` 주소(`argc[0]` 의 주소)를 가리키게 하고, `%rdi`를 `argc`로 설정

5. 마지막으로 가짜 "리턴 어드레스"를 푸시. Entry 함수는 절대 리턴되지 않겠지만, 해당 스택 프레임은 다른 스택 프레임들과 같은 구조를 가져야 함.

### Argument Parsing 예시

아래의 표는 스택과 관련 레지스터들이 유저 프로그램이 시작되기 직전에 어떤 상태인지를 보여줌. 스택은 아래 방향으로 커진다는 사실, 염두에 두자.

| Address | Name | Data | Type | 
|:---:|:---:|:---:|:---:|
| 0x4747fffc | argv[3][...] | 'bar\0' | char[4] |
| 0x4747fff8 | argv[2][...] | 'foo\0' | char[4] |
| 0x4747fff5 | argv[1][...] | '-l\0' | char[3] |
| 0x4747ffed | argv[0][...] | '/bin/ls\0' | char[8] |
| 0x4747ffe8 | word-align | 0 | uint8_t[] |
| 0x4747ffe0 | argv[4] | 0 | char * |
| 0x4747ffd8 | argv[3][...] | 0x4747fffc | char * |
| 0x4747ffd0 | argv[2] | 0x4747fff8 | char * |
| 0x4747ffc8 | argv[1][...] | 0x4747fff5 | char * |
| 0x4747ffc0 | argv[0] | 0x4747ffed | char * |
| 0x4747ffb8 | return address | 0 | void (*) () |

RDI: 4 | RSI: 0x4747ffc0

## Argument Passing 구현

PintOS `process_exec()`함수는 새로운 프로세스들에 인자를 전달하는 것을 지원하지 않고 있다. 이 함수를 확장 구현해서 지금처럼 단순히 프로개름 파일 이름만을 인자로 받아오게 하는 대신 공백을 기준으로 여러 단어를 나누어지게 만들어야 한다.

첫 번째 단어는 프로그램 이름, 두번째 단어는 첫 번째 인자, 이런 식으로 계속 이어지게 만들면 된다.

`strtok_r()`함수를 통해 구현하면 쉬울 것이다.

그래서 나는 아래와 같이 구현했다.

~~~c
int
process_exec (void *f_name) {
	char *str = f_name;
	char *save_ptr, *token;
	char *argv[MAX_ARGS];
	int argc = 0;
	bool success;

	token = strtok_r(str, " ", &save_ptr);
	while (token != NULL && argc < MAX_ARGS)
	{
		argv[argc++] = token;
		token = strtok_r(NULL, " ", &save_ptr);
	}

	char *file_name = argv[0];
	
	/* We cannot use the intr_frame in the thread structure.
	 * This is because when current thread rescheduled,
	 * it stores the execution information to the member. */
	struct intr_frame _if;

	_if.ds = _if.es = _if.ss = SEL_UDSEG;
	_if.cs = SEL_UCSEG;
	_if.eflags = FLAG_IF | FLAG_MBS;

	/* We first kill the current context */
	process_cleanup ();

	/* And then load the binary */
	success = load (file_name, &_if);

	// 여기서부터 내가

	if (success)
	{
		_if.R.rdi = argc;
		void *arg_addr[MAX_ARGS];
		void *start = _if.rsp;

		for(int i = argc - 1; i >= 0; i--)
		{
			size_t arg_size = strlen(argv[i]) + 1;
			start -= arg_size;
			memcpy (start, argv[i], arg_size);
			arg_addr[i] = start;
		}

		uintptr_t align = (uintptr_t)start % 16;
		if (align !=0)
			start -= align;

		start -= sizeof(char *);
		*(char **)start = NULL;

		for (int i = argc -1; i >= 0; i--)
		{
			start -= sizeof(char *);
			*(char **)start = arg_addr[i];
		}

		_if.R.rsi  = start;

		start -= sizeof(void *);
		*(void **)start = NULL;

		_if.rsp = start;
	}

	// hex_dump(_if.rsp, _if.rsp, USER_STACK - _if.rsp, true);

	// 여기까지 내가

	/* If load failed, quit. */
	palloc_free_page (file_name);
	if (!success)
		return -1;

	/* Start switched process. */
	do_iret (&_if);
	NOT_REACHED ();
}
~~~