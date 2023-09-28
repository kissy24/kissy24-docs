# 【Zig】stdライブラリのlogを散歩する

## はじめに

`log.zig` を読んで、使い方を理解するだけのお話です。

## 環境

- MacOS 11.6.6
- Zig 0.9.1

## 読んでみましょう

ソースは[こちら](https://github.com/ziglang/zig/blob/master/lib/std/log.zig)です。

### 1. 最初の70行程度

コメントが書かれていますね。要約するとこんな感じです。

1. ロギングを可能にするライブラリです。
2. スコープの設定がないと`.default`でセットされる。
    - 後述します。
3. std.log.scoped 関数で任意のスコープパラメータを設定できる。
    - scopedで説明します。
4. root.logで上書きする方法について。
    - root: 平易に言うと**実行ファイル**のこと(main.zigとか)
    - root.log: 実行ファイルのlog関数のこと

2~4については後述します。

### 2. ログレベルやスコープレベルの処理

ここで覚えておきたい点は2点ありますね。

- log_levelはrootで宣言したものが優先される
  
  ```zig
  pub const level: Level = if (@hasDecl(root, "log_level"))
      root.log_level
  else
      default_level;
  ```

- scope_levelsもrootで宣言したものが優先される

  ```zig
  const scope_levels = if (@hasDecl(root, "scope_levels"))
      root.scope_levels
  else
      [0]ScopeLevel{};
  ```

例えば、`main.zig`で`log_level`を下記のように宣言すると

```zig:main.zig
pub const log_level: std.log.Level = switch (builtin.mode) {
    .Debug, .ReleaseSafe => .debug,
    .ReleaseFast, .ReleaseSmall => .err,
};

pub fn main() anyerror!void {
    std.log.debug("All your codebase are belong to us.", .{});
}
```

info ではなく、 debug でログが出力されます。

```sh
$ zig build run

debug: All your codebase are belong to us.
```

scope_levelsの変更例も挙げます。

```zig:main.zig
pub const scope_levels = [1]std.log.ScopeLevel{
    std.log.ScopeLevel{ .scope = .default, .level = .err },
};

pub fn main() anyerror!void {
    std.log.info("All your codebase are belong to us.", .{});
}
```

本来であれば、infoレベルの結果が返されますが、errレベルの結果のみを返すように変更したため、下記のように何も返りません。

```sh
$ zig build run
```

### 3. log関数の処理

気になった処理に絞って説明します。

```zig
const effective_log_level = blk: {
    inline for (scope_levels) |scope_level| {
        if (scope_level.scope == scope) break :blk scope_level.level;
    }
    break :blk level;
};
```

上記は、先程のscope_levelの上書きがあるかどうかを判定しています。

```zig
...
    if (@hasDecl(root, "log")) {
        if (@typeInfo(@TypeOf(root.log)) != .Fn)
            @compileError("Expected root.log to be a function");
        root.log(message_level, scope, format, args);
    } else {
        defaultLog(message_level, scope, format, args);
    }
```

ここが、先ほどのroot.logを上書きする処理になります。  
上書き例を下記に記載します。

```zig:main.zig
pub fn log(
    comptime level: std.log.Level,
    comptime scope: @TypeOf(.EnumLiteral),
    comptime format: []const u8,
    args: anytype,
) void {
    const scope_prefix = "(" ++ switch (scope) {
        .my_project, .nice_library, .default => @tagName(scope),
        else => if (@enumToInt(level) <= @enumToInt(std.log.Level.err))
            @tagName(scope)
        else
            return,
    } ++ "): ";

    const prefix = "[" ++ level.asText() ++ "] " ++ scope_prefix;

    std.debug.getStderrMutex().lock();
    defer std.debug.getStderrMutex().unlock();
    const stderr = std.io.getStdErr().writer();
    nosuspend stderr.print(prefix ++ format ++ "\n", args) catch return;
}

pub fn main() anyerror!void {
    std.log.info("All your codebase are belong to us.", .{});
}
```

`defaultLog`と異なるフォーマットのログが出力されます。

```sh
$ zig build run
[info] (default): All your codebase are belong to us.
```

`defaultLog`については解説しませんが、root.logで上書きする際に参考になるので一読することをおすすめします。

### 4. scoped関数の処理

面白い書き方だなと思いました。例えば、スコープをmainとした時に、  
`std.log.scoped(.main).info("hoge", .{});`  
となり、関数型の組み上げ方を意識されているのかなと思いました。

こちらも気になった処理について説明します。

```zig
pub const emerg = @compileError("deprecated; use err instead of emerg");
pub const alert = @compileError("deprecated; use err instead of alert");
pub const crit = @compileError("deprecated; use err instead of crit");
...
pub const notice = @compileError("deprecated; use info instead of notice");
```

うーん、他言語使いへの配慮ですかね…？インテリセンスが整ってきたら逆に邪魔になるような気がしてしまうんですが、どうなんですかね？

```zig
pub fn info(
    comptime format: []const u8,
    args: anytype,
) void {
    log(.info, scope, format, args);
}
```

基本的に、scopedの戻り値はこれらのstructになります。
実際にスコープを変えてみましょう。

```zig:main.zig
pub fn main() anyerror!void {
    const hoge_log = std.log.scoped(.hoge);
    hoge_log.info("All your codebase are belong to us.", .{});
}
```

結果は下記のようになります。

```sh
$ zig build run
info(hoge): All your codebase are belong to us.
```

ちなみに、ここの`.hoge`に相当する`@Type(.EnumLiteral)`は「タグ値が名前に対応しない場合、安全が確認された未定義の動作を呼び出す」らしい。つまり、誤字っても通るので注意。(詳細は[tagName](https://ziglang.org/documentation/master/#tagName)からどうぞ)

### 4. default設定

スコープの設定がないと`.default`でセットされる話をします。

```zig
pub const default = scoped(.default);
...
pub const info = default.info;
```

このように各種ログレベルはscopedに変更がない場合、`.default`という未定義のタグ値が勝手に入ります。個人的に、defaultって何？って感じなのでもっと分かりやすい命名にならないかなと思っています。

## おわりに

Zigのコードを読むのは、今回が初めてだったのですがなかなか読みやすいですね。  
ただ、logの機能としては他言語に比べると弱いなと感じました。  
moduleのトレースやdatetimeなどが私が探した限りなかったので、handlerやformatterとしての機能が不十分に感じられました。
今後に期待しています。
