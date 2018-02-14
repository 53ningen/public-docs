GNU-Coreutils touch コマンドの実装を読む
===

　基本的に `utime(2)` を使えば良さそうなので、実装が楽そうだし、自分で書いて見つつ、GNU-Coreutils の実装を眺めてみるかなという気持ちになったので挑戦。`utime(2)` は次のようなインターフェースを持っている。

```c
#include <sys/types.h>
#include <utime.h>

int utime(const char *path, struct utimbuf *buf);

struct utimbuf {
    time_t actime;
    time_t modtime;
}
```

### 基本的な実装

単純に actime と modtime を更新してやればよいので、次のように書けば良さそう

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <time.h>
#include <utime.h>

int main(int argc, char *argv[])
{
    if (argc != 2) {
        fprintf(stdout, "Usage: %s filename\n", argv[0]);
        exit(1);
    }

    if (utime(argv[1], NULL) < 0) {
        perror(argv[1]);
        exit(1);
    }
    exit(0);
}
```

さて、GNU-Coreutils の touch の実装はどうなっているのだろうか。答え合わせをしてみることにした。


### GNU-Coreutils の実装

これだと `-c` をつけたときの no_create モードの振る舞いになってしまうので、ちょっとだけ手を加えて、ファイルがなかったときには作成する振る舞いを追加してあげればだいたいあってるかなと思いつつ、ソースコードを覗きにいく。GNU-Coreutils のソースコードは[こちら](https://github.com/Coreutils/Coreutils/blob/master/src/touch.c#L122-L204)。実処理部は、短くてそんなに複雑なことやってないので読みやすい。

```c
if (STREQ (file, "-"))
  fd = STDOUT_FILENO;
else if (! (no_create || no_dereference))
  {
    /* Try to open FILE, creating it if necessary.  */
    fd = fd_reopen (STDIN_FILENO, file, O_WRONLY | O_CREAT | O_NONBLOCK | O_NOCTTY, MODE_RW_UGO);

    /* Don't save a copy of errno if it's EISDIR, since that would lead
       touch to give a bogus diagnostic for e.g., 'touch /' (assuming
       we don't own / or have write access to it).  On Solaris 5.6,
       and probably other systems, it is EINVAL.  On SunOS4, it's EPERM.  */
    if (fd == -1 && errno != EISDIR && errno != EINVAL && errno != EPERM)
      open_errno = errno;
  }
```

まずは、ファイルディスクリプタの取得から、標準入力かそうでないかみたいな分岐。普通にファイルから読むときは、ファイルがなかったら作成も行う。`no_create` は `-c` オプション時にフラグが立つ。また `no_dereference` は、`-h` オプション。それぞれのオプションは次のように定義されている。

```c
/* (-c) If true, don't create if not already there.  */
static bool no_create;

/* (-h) If true, change the times of an existing symlink, if possible.  */
static bool no_dereference;
```

続いて細々とした調整。どの time を書き換えますかみたいな話。`-a`, `-m`, `--times` オプションで色々いじれる。詳細は man をみてください。

```c
/* Bitmasks for `change_times'. */
#define CH_ATIME 1
#define CH_MTIME 2

/* Which timestamps to change. */
static int change_times;

if (change_times != (CH_ATIME | CH_MTIME))
  {
    /* We're setting only one of the time values.  */
    if (change_times == CH_MTIME)
      newtime[0].tv_nsec = UTIME_OMIT;
    else
      {
        assert (change_times == CH_ATIME);
        newtime[1].tv_nsec = UTIME_OMIT;
      }
  }

if (amtime_now)
  {
    /* Pass NULL to futimens so it will not fail if we have
       write access to the file, but don't own it.  */
    t = NULL;
  }
```

実処理部はこんな感じになってて、`utime(2)` を直接呼び出しているわけではなかった。

```c
ok = (fdutimensat (fd, AT_FDCWD, (fd == STDOUT_FILENO ? NULL : file), t,
                   (no_dereference && fd == -1) ? AT_SYMLINK_NOFOLLOW : 0)
      == 0);
```

というわけで [fdutimensat の実装](https://github.com/Coreutils/gnulib/blob/master/lib/fdutimensat.c)をみにいくとこんな感じ。

```c
int
fdutimensat (int fd, int dir, char const *file, struct timespec const ts[2],
             int atflag)
{
  int result = 1;
  if (0 <= fd)
    result = futimens (fd, ts);
  if (file && (fd < 0 || (result == -1 && errno == ENOSYS)))
    result = utimensat (dir, file, ts, atflag);
  if (result == 1)
    {
      errno = EBADF;
      result = -1;
    }
  return result;
}
```

ファイルに対してはさらに `utimensat(2)` を呼んでいるようなので、man をみる。

```c
utimensat, futimens - change file timestamps with nanosecond precision

#include <fcntl.h> /* Definition of AT_* constants */
#include <sys/stat.h>

int utimensat(int dirfd, const char *pathname,
              const struct timespec times[2], int flags);

int futimens(int fd, const struct timespec times[2]);

DESCRIPTION
       utimensat() and futimens() update the timestamps of a file with nanosecond precision.  This contrasts with the historical utime(2) and utimes(2), which permit only second and microsecond pre‐
       cision, respectively, when setting file timestamps.
```

なるほど。`utime(2)` と比較して `utimensat(2)` はナノ秒単位での指定が可能なのですね。というわけで GNU-Coreutils の `touch` の内部では、`utimensat(2)` が呼ばれていることがわかりました。ちなみに strace をよべば、ソースコード読まなくても普通にわかる。

```sh
$ strace touch hoge
execve("/usr/bin/touch", ["touch", "hoge"], [/* 21 vars */]) = 0
brk(0)                                  = 0x1885000
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fd4facb7000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=23314, ...}) = 0
mmap(NULL, 23314, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fd4facb1000
close(3)                                = 0
open("/lib64/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0@\34\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=2118128, ...}) = 0
mmap(NULL, 3932672, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fd4fa6d8000
mprotect(0x7fd4fa88e000, 2097152, PROT_NONE) = 0
mmap(0x7fd4faa8e000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1b6000) = 0x7fd4faa8e000
mmap(0x7fd4faa94000, 16896, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fd4faa94000
close(3)                                = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fd4facb0000
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fd4facae000
arch_prctl(ARCH_SET_FS, 0x7fd4facae740) = 0
mprotect(0x7fd4faa8e000, 16384, PROT_READ) = 0
mprotect(0x60d000, 4096, PROT_READ)     = 0
mprotect(0x7fd4facb8000, 4096, PROT_READ) = 0
munmap(0x7fd4facb1000, 23314)           = 0
brk(0)                                  = 0x1885000
brk(0x18a6000)                          = 0x18a6000
brk(0)                                  = 0x18a6000
open("/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=106070960, ...}) = 0
mmap(NULL, 106070960, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fd4f41af000
close(3)                                = 0
open("hoge", O_WRONLY|O_CREAT|O_NOCTTY|O_NONBLOCK, 0666) = 3
dup2(3, 0)                              = 0
close(3)                                = 0
utimensat(0, NULL, NULL, 0)             = 0
close(0)                                = 0
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```
