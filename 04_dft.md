# 密度汎関数理論

密度汎関数理論（DFT; density functional theory）を使ってみます。
現代の量子化学計算のうち、少なくとも8割はDFTを用いた計算と言っても良いでしょう。
以下は量子化学計算としてのDFTについてですが、DFT自体は量子化学計算だけで用いられるわけではなく、物理分野で凝集系を取り扱う際にも用いられます。
DFT自体が「多体系の全ての物理量は空間的に変化する電子密度の汎関数（すなわち関数の関数）として表され」（[Wikipedia](https://ja.wikipedia.org/wiki/%E5%AF%86%E5%BA%A6%E6%B1%8E%E9%96%A2%E6%95%B0%E7%90%86%E8%AB%96)より）るという理論を用いているだけなので。
また、工学系や数学の人と話す際はdiscrete Fourier transformではないと断るようにしましょう。

Hartree--Fock法は、（定義に一つとしての）電子相関が含まれていません。
そこで、伝統的な波動関数理論では、Hartree--Fock法の計算のあとで、励起した電子配置を考慮することによって電子相関を含めています。
しかし、DFTの場合は「交換相関汎関数」という謎の汎関数を用いることにより電子相関と交換相互作用を計算しています。
このため、この（交換相関）汎関数の質によって計算結果の質が大きく変わってきます。
それでも、波動関数理論と比べて追加の計算があまり必要ないことと、汎関数が上手に設計されていることもあり、低コストで高い精度の計算ができる傾向があります。
しかも、一般的にはHartree--Fock法のように単配置の計算（スレーター行列式を一つしか使わないという意味）なので、入力ファイルの作成も簡単です。

というわけで、DFTの良い点としては、

- 割と高速
  - Hartree--Fock法と大体同じ（実装にもよる）
- 割と高精度
  - B3LYPという汎関数が特に信頼されています
- 計算のセットアップが簡単
  - Hartree--Fock法の入力ファイルから少し変更するだけ

昨今では実験を専門とする方も量子化学計算を用いていますが、99%はDFTの計算でしょう。
Gaussianという量子化学計算ソフトウェアを用いると、簡単にできます。
そのようなこともあり、実験を専門とする方と話すときは、「計算化学 = DFT」と考えても良いでしょう。

一方、DFTの良くない点としては、

- 汎関数依存性が厄介
  - 演習問題で確かめましょう
- 系統性がなさそう
  - full CIに近づく方法はありそうですが、full CIに至る方法は現状不明
- 多配置性・多参照性を持つ系は難しそう
  - Hartree--Fock法と同様に、一つの行列式で記述する方法のため
- 分散力、エネルギーギャップ、自己相互作用など
  - 勉強してください
  - とは言え汎関数次第

必ずDFTがうまくいくというわけではありませんが、DFTでうまくいかない系は「例外」として、
より高度な計算（電子相関理論）を行うというスタンスの人が多いのではないでしょうか（私見）。
「高度な計算を行う」場合でも、その前段階として構造最適化をする時にDFTを用いる場合も良くあります。
上記の点も含めた良くない点は、自分で使ったり学会での発表を聞くと、なんとなく雰囲気は伝わってくると思います。

日本語でDFTの導入的な書籍は、やはり「密度汎関数法の基礎」です。やや高いですが。
無料でアクセスできる中だと、英語ですが[ABC of DFT](https://dft.uci.edu/doc/g1.pdf)とかでしょうか。
京都大学からだと無料でアクセスできる電子ジャーナルもあるかもしれません（私が昔少し勉強した「A Chemist's Guide to Density Functional Theory」が無料でダウンロードできました）。
さらに発展的な書籍は「Density Functional Theory of Atoms and Molecules」（邦訳：原子・分子の密度汎関数法）が最も有名です。

歴史的には、1964年の「ホーヘンベルク・コーンの定理」によって密度汎関数理論により原子・分子の計算が可能であることが示されました。
さらに翌年1965年の「コーン・シャム理論」によって実際の計算手続きが示されました。
ただしそれが実用的であるかは別の話で、当時（LDA汎関数）は精度が悪かったとのことです。
そして1994年?のB3LYPの登場以降、その低い計算コストの割に精度が高いこともありDFTが広く普及してきました。
1998年のノーベル化学賞であるWalter Kohn先生のご受賞は「密度汎関数法の開発」によるものです。
ちなみにHartree--Fock法は1930年代あたりの方法で、1950年辺りから計算機の発達により活発に用いられてきました。

## 計算例

B3LYPという汎関数を用いて計算を行ってみます。

```python
  1 from pyscf import gto, dft
  2 mol = gto.M(atom="H 0 0 0; H 0 0 1.2", basis="ccpvdz", verbose=4)
  3 mf = dft.RKS(mol)
  4 mf.xc = 'b3lyp'
  5 mf.kernel()
```

上記計算の計算レベルは「B3LYP/cc-pVDZ」と書きます。一般的には汎関数の名前をスラッシュの前に書きます。
Hartree--Fockの場合から変更した点は、1行目の`scf`を`dft`にした点、
3行目の`scf.RHF`を`dft.RKS`にした点、そして4行目に`mf.xc = 'b3lyp'`を追加した点です。
ここで`RKS`は、"restricted Kohn--Sham"を意味しています。
狭義のDFTとして、Kohn--Sham法とも呼ばれています。

`mf.xc = 'b3lyp'`が汎関数を指定しています。
他にも代表的な汎関数として、適当に挙げさせていただきますと、

- local density approximation (LDA)
  - さすがに使わないと思います。
- generalized gradient approximation (GGA)
  - BLYP
  - PBE
- hybrid
  - B3LYP
  - PBE0
- meta-GGA
  - M05/M06(L/2X)
- range-separated / long-range corrected
  - LC-BLYP
  - CAM-B3LYP
  - ωB97X-D
- double-hybrid
  - B2PLYP

とかがありますが、詳細は[Gaussianのページ](https://gaussian.com/dft/)などから勉強してください。
一応下に行くほど「ヤコビのはしご（？）」だったか基準でレベルが高くなり、Full CIに近づけるとか、そんなことが言われている気がします（何も調べずに記憶だけで書いています）。
PySCFがどこまでサポートしているかは別の話で、私も試せていません。

出力ファイルの見方に関しては、基本的には[Hartree--Fockの場合と同じ](02_sp_output.md)です。
ですが、以下の行だけは確認しておきましょう。

```
 53 XC functionals = b3lyp
```

XC functionals (XC: exchange--correlation)は日本語では前述の「交換相関汎関数」と呼ばれており、
DFTの枠組みで電子相関と相関相互作用をどのように取り入れるかを指定するためのキーワードです。
ここでは入力ファイルで指定したとおり、`B3LYP`という汎関数を用いた計算を行っています。
相対エネルギーを計算するときは、必ず同じ汎関数との間で比較する必要があります。

他の汎関数を用いるためには、`mf.xc`で他の汎関数の名前を指定します。
本来は交換汎関数と相関汎関数を露わに指定する必要がありますが、PySCFではよく使われる汎関数に対してエイリアスが存在するため、特に何も考えなくても単語を一つ入力することで汎関数を指定できます。
エイリアスについては、[libxc.pyのソースコード](https://github.com/pyscf/pyscf/blob/master/pyscf/dft/libxc.py)を参照してください。
`XC_CODES.update`や`XC_ALIAS`の配列の左側の単語(?)を`mf.xc`のところで入力すれば、B3LYP以外の汎関数を使えると思います。

## 分散力補正

***ライブラリとの結合がうまくできないかも。というか、昨年度はできませんでした。***

上の方で分散力（dispersion）に問題があると書きました。
汎関数によっては補正されていますが、*posteriori*（後天的・経験的）に補正することができます。
私の環境では、ライブラリが見つからずに実行できなかったので、面倒になってやめました。
[マニュアル](https://pyscf.org/user/dft.html)を参考にしてやってみてください。

実行できるか確認できていませんが、
```python
  1 from pyscf import gto, dft, dftd3
  2 mol = gto.M(atom="H 0 0 0; H 0 0 1.2", basis="ccpvdz", verbose=4)
  3 mf = dftd3(dft.RKS(mol))
  4 mf.xc = 'b3lyp'
  5 mf.kernel()
```
とかすると良いのではないでしょうか。

こちらも様々な種類の補正（D2, D3, D3(BJ)）があります。
おそらくここではD3をつかっていると思います。

## 演習問題

様々な手法（HF/DFT）を用いて計算してみましょう。
汎関数（できるだけ複数）・基底関数は自分で選びましょう。
[前回](03_geomopt.md)ご紹介した構造最適化も利用できます。
~~また、必要に応じて分散力補正を用いましょう。~~
時間がある人は、MP2やcoupled cluster、紹介していない手法も使ってみてください（軌道エネルギー除く）。

- 適当な分子の軌道エネルギー（特にHOMO--LUMOギャップ）を計算してみましょう。
計算手法や汎関数によって、どのような傾向が見られるでしょうか。

- ネオン二原子の結合（相互作用）エネルギーを計算してみましょう。
CCSD(T)の結果からは、3.0から3.2 Angstromあたりに、-0.09 kcal/molぐらいの極小値があるとのことです。
HFやDFTで再現できるでしょうか。

- 水分子二量体のO-O長さと結合エネルギーを計算してみましょう。
[実験値](https://en.wikipedia.org/wiki/Water_dimer)は、それぞれ2.98 Angstrom、3.16 kcal/molと言われています。
一量体と二量体（どのような構造・配向が安定になるか？）で構造最適化を行い、そうして得られたエネルギーを用いて結合エネルギーを計算します。

- [前回](03_geomopt.md)計算したディールス・アルダー反応を、汎関数を変えていくつか計算し、汎関数依存性を調べてみましょう。