---
title: Compiled
summary: TryHackMe reverse-engineering room using Ghidra to inspect a Linux binary, understand scanf input parsing, and recover the required password.
date: 2026-07-02
tags: [TryHackMe, Reverse Engineering, Ghidra, Binary Analysis, CTF, Purple Team]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/compiled
---

# CTF Room: Compiled
- [Link to room](https://tryhackme.com/room/compiled)
- **Difficulty:** Easy
- **Category:** Reverse Engineering, Binary Analysis, Purple Team
- **OS:** Linux

## 1. Brief
Compiled is a short TryHackMe reverse-engineering room. The room gives us a binary and asks for the password.

The note says the binary will not execute properly in the AttackBox, but we can still solve it. That tells me not to get stuck trying to run it. Strings may help a bit, but this one needs a decompiler.

Tools used:

- `strings`
- Ghidra

## 2. First Look
I started by running the binary locally to see the behaviour.

### Command
```bash
./Compiled.Compiled
```

### Output
```text
Password: test
Try again!
```

No surprise there. It asks for a password and rejects the test input.

## 3. Ghidra
I loaded the binary into Ghidra and checked `main`. The decompiled logic was small enough to follow without going deep into assembly.

### Decompiled Logic
```c
undefined8 main(void)
{
  int iVar1;
  char local_28 [32];

  fwrite("Password: ", 1, 10, stdout);
  __isoc99_scanf("||DoYouEven%sCTF||", local_28);

  iVar1 = strcmp(local_28, "__dso_handle");
  if ((-1 < iVar1) && (iVar1 = strcmp(local_28, "__dso_handle"), iVar1 < 1)) {
    printf("Try again!");
    return 0;
  }

  iVar1 = strcmp(local_28, "||_init||");
  if (iVar1 == 0) {
    printf("Correct!");
  }
  else {
    printf("Try again!");
  }

  return 0;
}
```

The important line is the `scanf` format string:

```c
__isoc99_scanf("||DoYouEven%sCTF||", local_28);
```

That means the program expects input to start with `DoYouEven`, then it captures the `%s` part into `local_28`.

The next useful comparison checks whether `local_28` equals `||_init||`.

## 4. Working Out The Input
To make the program print `Correct!`, I need the captured `%s` value to be:

```text
||_init||
```

Since the literal prefix in the `scanf` format is `DoYouEven`, the full input needs to be that prefix plus the captured value.

### Input
```text
||DoYouEven_init||
```

That makes `local_28` equal `||_init||`, which passes the final `strcmp`.

## 5. Answer

#### What is the password?
### Answer
```text
||DoYouEven_init||
```

## 6. Summary
This room is a neat reminder that `strings` only gets you so far. The interesting part was not a hidden hardcoded password by itself, it was how `scanf` carved the input apart before the `strcmp`.

Ghidra made the control flow clear: avoid the wrong comparison, make the captured string equal `||_init||`, and build the final password from the `scanf` prefix.
