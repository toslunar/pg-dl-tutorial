# 1. 学習対象のモデルを定義する

はじめに学習対象のモデルを定義します。

パラメータで特徴付けられるモデルをパラメトリックモデルとよび，本チュートリアルでは
パラメトリックモデルを対象にします。

パラメトリックモデルはパラメータ $\theta$ で特徴付けられた関数 $y=F(x; \theta)$ で表すことができます。
この関数 $F(x; \theta)$ は $x$ を受け取り， $y$ を返すような関数です。
この関数の挙動がパラメータ $\theta$ で変わることを示すために，引数とは違って $;\theta$ と表します。

例えば，線形関数（またはアフィン変換）であるモデルは

```math
\begin{align}
f(x; θ) &= Wx + b \\
θ &= (W, b)
\end{align}
```

のように表すことができます。
この関数は，パラメータである $W$ や $b$ を変えることでその挙動を変えることができます。

Chainerではこのようなパラメータがひも付いた関数をLinkとよびます。

ディープラーニングで利用される代表的なLinkは `chainer.links` でサポートされています。
また，自分で新しいLinkを作ることができます。

以降では，この `chainer.links` を `L` として使えるようにします。

```
from chainer import links as L
```

例えば，先程の線形変換は `Linear` という名前のLinkとして用意されており，例えば入力が100次元，
出力が20次元の線形変換を表す `Linear` は次のように作ることができます。

```
lin = L.Linear(100, 20)
```

この `lin` はLinkオブジェクトですが，次のように関数呼び出しをすることができます。
（この関数呼び出しは `__call__` で定義されおり，演算子オーバーロードで実現されています。）

```
import numpy as np
from chainer import Variable
...
x = Variable(np.ones((10, 100), dtype=np.float32))
y1 = lin(x)
```

この `numpy` ， `Variable` については後で詳しく説明します。
`np.ones((10, 100), dtype=np.float32)` は10行100列で全ての値が1であるような行列を作ります。
`Variable` はその値に加えて学習に必要な情報が埋め込まれているオブジェクトとだけ覚えてください。
つまりここでは10個の100次元のベクトルを用意し，それをVariableというオブジェクトにセットし，
それを `lin` の引数として与えて，出力を `y` として計算しています。

`Variable v` に格納されている値は `v.data` として参照できます。
例えば，上の例の入力と出力は

```
print(x.data)
print(y1.data)
```

で調べることができます。
example.pyでは表示スペースの関係で行列のサイズを小さくしていますが，
数値を変えて実験してみてください。

パラメータがひも付いた関数Linkと違い，パラメータで特徴づけられていない関数はFunctionとよびます。
ディープラーニングで利用されている代表的な関数は `chainer.functions` で定義されています。
また，自分で新しいFunctionを作ることができます。

以降では，この  `chainer.functions` をFとして使えるようにします。

```
from chainer import functions as F
```

例えば，ディープラーニングでよく使われるReLUとよばれる非線形関数 $f_{relu}$ は

$$f_{relu}(x)=max(x,0)$$

で定義されますが，Chainerでは次のように記述できます。

```
from chainer import functions as F
...
y2 = F.relu(x)
```

これらのLinkとFunctionを組みわせて複雑な関数を作ることができます。
例えば，線形変換を適用した後にReLUを適用した結果は次のように計算されます。

```
y3 = F.relu(lin(x))
```

## 多層パーセプトロンの例

（注：ここは飛ばしても構いません）

これらを組み合わせて多層パーセプトロンをChainerを用いて実装したのが次の例です。

```
class MLP(chainer.Chain): # MultiLayer Perceptron

    def __init__(self, n_units, n_out):
        super(MLP, self).__init__(
            # the size of the inputs to each layer will be inferred
            l1=L.Linear(None, n_units),  # n_in -> n_units
            l2=L.Linear(None, n_units),  # n_units -> n_units
            l3=L.Linear(None, n_out),  # n_units -> n_out
        )

    def __call__(self, x):
        h1 = F.relu(self.l1(x))
        h2 = F.relu(self.l2(h1))
        return self.l3(h2)
```

各機能は今後詳細に説明されますので，ここでは概要だけ説明します。
理解できなくてもそのまま飛ばして問題ありません。

`MLP` のコンストラクタでは入出力層と中間層を定義しています。
ここではすべて `L.Linear` を使っています。
`L.Linear` の第一引数には `None` を指定することで実際の入力からユニット数を自動で設定してくれます。

`__call__` では先ほど定義した層に入力を与えて計算（順計算）を行います。
まず `l1` に大元の入力 `x` を与え，それを線形変換したものにReLUを適用します。
その計算結果 `h1` を次の層 `l2` に与え同様の計算を行います。
`l3` に関しても同様に前層の結果を元に計算を行います。
最終的に `l3` の結果を返すことで計算が完了します。

ここで，ニューラルネットワークをご存じの方は逆方向の計算が記述されていないことに気付いたかもしれません。
Chainerは順計算を記述するだけで自動的に逆方向の計算を用意します。
この仕組みに関しても後ほど仕組みを説明します。
このページでの説明はここまでに止めます。

## 課題

3行5列のランダムな値で埋められた行列に対して，上の例の `lin` を適用した後に ReLU を適用し，その結果を表示してください。
なお，3行5列のランダムな値は `np.random.rand(3, 5).astype(np.float32)` で作ることができます。
example.pyのコードを変更していたら，列数を `lin` の入力と合うように適切に変更してください。
