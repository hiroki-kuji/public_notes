# Dual-species neutral-atom quantum computing (NAQC)

## 要旨

Dual-species NAQC では、data qubit / ancilla qubit / measurement / reset / networking interface などの役割を別の原子種に分担させることで、単一種では避けにくい readout crosstalk や mid-circuit operation の overhead を減らすことが狙いになる。

特に重要なのは、species selectivity を ancilla/data 分離の物理的リソースとして使う点である。ancilla を別元素にしてしまえば、ancilla の readout light が data qubit にほとんど共鳴せず、single-species で必要になりがちな shelving / hiding / transport を減らせる可能性がある。

この路線は、少なくとも Beterov–Saffman (2015) の段階で、Rb/Cs の異種原子 Rydberg 相互作用と low-crosstalk QND measurement の可能性が理論的に明示されたところから始まったと見てよい。

その後の流れは、概ね

1. 異種原子相互作用と QND 測定の理論的動機づけ
2. dual-species / mixed-species array の実装
3. mid-circuit readout・feed-forward・continuous reload などの運用技術
4. interspecies entangling gate と QND measurement の実証
5. QEC に向けた stabilizer / syndrome measurement への接続

という順で進んできた。

ここでいう QND measurement は、単に atom loss が少ないという意味ではなく、測りたい観測量が repeatable で、measurement backaction が制御され、repeated syndrome extraction に使える測定を指す。測定対象の量子状態全体を保存する、という意味ではない。

---

## 1. Why dual-species?

NA QEC のボトルネックの一つは、ancilla を mid-circuit で読んだときに data qubit をどう守るかである。

single-species architecture では、主に次のような方法が使われる。

1. measurement/entangling zone に ancilla を運ぶ（zoned / shuttling architecture）
2. data を shelving / hiding して、ancilla だけ読む
3. site-selective / state-selective readout を工夫する

これらはどれも有効だが、論理サイクル時間、制御の複雑さ、追加エラー、shuttling の遅さなどのコストがある。

dual-species の発想は、ancilla を別元素にしてしまえば、ancilla の readout light が data にほとんど共鳴しない。つまり、種選択性 (species selectivity) を ancilla/data 分離の物理的リソースとして使う。

重要なのは、dual-species は single-species の代替というより、single-species では外付けだった補助操作をハードウェアに埋め込む発想だということになる。読み出しのクロストーク抑制、ancilla の再初期化、lost atom の補充、photonic interface の分離などに向いている？

---

## 2. History

### 2.1 理論： 異種原子相互作用と QND の発想

**Beterov & Saffman, PRA 92, 042710 (2015)**
*Rydberg blockade, Förster resonances, and quantum state measurements with different atomic species*

Rb と Cs の異種原子間の Rydberg–Rydberg 相互作用を系統的に計算し、その強い Förster 共鳴を使えば、長距離エンタングルメント生成や、低クロストークな QND 測定が可能になることを示した。
また、Rb と Cs の異種間相互作用強度を評価し、加えて Rb-Rb / Cs-Cs でも異なる主量子数を使った場合の強い Förster resonance を整理した。
この論文で示している中心成果は、interaction strength の計算とmeasurement scheme の提案であって、実際の異種間 gate や syndrome measurement の実証までは行っていない。

---

### 2.2 実験: dual-species trapping の装置実証

**Singh et al., PRX 12, 011040 (2022)**
*Dual-Element, Two-Dimensional Atom Array with Continuous-Mode Operation*


Rb/Cs の dual-element tweezer array を実装し、最大 512 trapping sites で独立な loading / cooling / control / measurement を実証した最初の基盤実証。
Rb 用 811 nm tweezer と Cs 用 910 nm tweezer を組み合わせ、最大 512 サイトの 2D 配列で、両元素を独立にロード・冷却・イメージングできることを示した。surface-code を意識した interleaved geometry も作っている。
他方の MOT や tweezer を同時に使っても、各元素のロード効率は50%以上で安定し、元素間クロストークは小さい。さらに、片方を保持したまま他方を再ロードでき、50 分の連続運転で常時 115 個以上の原子を実験に使える continuous-mode operation を実証した。
同種原子 array の弱点だった readout crosstalk と reloading 時の停止時間を、二種原子化で避けられることを示した。
この論文で実証したのはtrap / load / image / continuous operationまで。著者ら自身は、次段階として Rb-C s 間 Rydberg gate、QND readout、deterministic rearrangement、QEC 応用を挙げており、まだこの時点では相互作用ゲートや syndrome measurement の実証ではない。

---

### 2.3 実験: dual-species による in-sequence control

**Singh et al., Science 380, 1265–1269 (2023)**
*Mid-circuit correction of correlated phase errors using an array of spectator qubits*


dual-species 中性原子アレイを使って、spectator qubit をノイズセンサーとして使う mid-circuit error suppression を実証した論文。二種原子 NAQC の流れでは、dual-species array を単なる low-crosstalk readout ではなく、実時間の誤差補正に使った最初の実装になる。
具体的にはCs spectator qubit で環境ノイズをその場でプローブし、in-sequence readout、classical data processing、feed-forward を回して、Rb data qubit の位相誤差を回路実行中に補正した。さらに著者らは、この実装が mid-circuit readout、real-time processing/feed-forward、coherent mid-circuit reloading という中性原子プロセッサ拡張の主要要素を与えると述べている。
相関した phase error を、回路の途中で抑制できることを示した。論文の主張は「完全な QEC を達成した」ではなく、correlated noise に対して spectator-based correction が有効であり、そのための mid-circuit ツール群が neutral-atom platform 上で動くことを示した。

まとめると,
- dual-species で crosstalk-free mid-circuit readout ができること、
- feed-forward まで含めた in-sequence operation が成立すること、
- species separation が「測定」だけでなく「制御構造」そのものに効くこと、
を示した。

dual-species の価値を、元素分離による readout のしやすさから一歩進めて、能動的な mid-circuit 誤差抑制まで拡張した論文。流れとしては、dual-species array の装置化 → spectator-based correction の実証 → 将来の syndrome measurement / QEC へ接続の中間にあるか。

---

### 2.4 実験: dual-species Rydberg array を実際に構築し、異種間相互作用と mid-circuit 制御の基本要素を実証.

**Anand et al., Nat. Phys. 20, 1744–1750 (2024)**
*A dual-species Rydberg array*

この論文が、Rb/Cs dual-species QEC 路線の最初の大きな実験的 milestone。Rb/Cs 二種原子 Rydberg array の本格的なプラットフォーム実証。dual-species を使って、異種間 blockade、状態転送、Bell 状態生成、QND 測定まで一通り示した。
具体的には、Rb/Cs 配列を実現し、Rydberg 状態の選択と電場制御により、van der Waals 相互作用領域と、未観測だった異種間 Förster resonance に由来する resonant dipole–dipole 相互作用領域の両方にアクセスした。また、その相互作用を使って、interspecies Rydberg blockade、species 間の量子状態転送、異種間 controlled-phase gate による Rb–Cs Bell state 生成、さらに Cs 補助 qubit を用いた Rb qubit の QND 測定を実証した。

主な成果:
- dual-species Rydberg array の実現
- interspecies Förster resonance による enhanced interaction
- interspecies blockade
- interspecies state transfer
- interspecies CZ gate による Bell state 生成
- auxiliary-based QND measurement を without the need for qubit transport で実証

ただし数値的には、
- Bell fidelity: 0.69(3)
- QND readout fidelity: 0.76(2)
- QND-ness: 0.94(2)
で、まだ低い。

---

### 2.5 理論: dual-soeciesでFast measurements and multiqubit gates.

**Petrosyan et al., PRA 110, 042404 (2024)**
*Fast measurements and multiqubit gates in dual-species atomic arrays*

Rb/Cs dual-species を前提に、surface-code 的な syndrome measurement を高速化する具体的提案。また、Rb data qubit と Cs ancilla qubit の役割分担を前提に、異種間 multiqubit gate を使う測定方式を提案している。
具体的には、1 個の Cs ancilla を $k≥1$ 個の Rb qubit と結びつける inter-species CNOT_k を中核に据え、異種間・同種間で異なる Rydberg 相互作用強度を利用して、fast syndrome measurement を設計した。これによりCs data/ancilla と Rb measuring atoms を組み合わせ、few auxiliary atoms で measurement fidelity > 0.9999、integration time < 5 μs を想定している。これまでの単なる low-crosstalk readout ではなく、高速 syndrome measurement と multiqubit gate というQEC 指向の理論研究。

---

### 2.6 実験: 高 fidelity な異種間 gate と syndrome measurement

**Miles et al., arXiv:2603.13492 (2026)**
*Qubit syndrome measurements with a high fidelity Rb-Cs Rydberg gate*

高 fidelity の Rb–Cs gate を実際に動かして、dual-speciesの実用化を進めた論文。
具体的には,個別にアドレス可能な Rydberg ビームと独立な種選択 readout を備えた Rb/Cs array を組み、SU(2) randomized benchmarking で異種間 gate を評価したうえで、2 原子 QND 測定と、surface code 境界の weight-2 parity check に対応する 3 原子 plaquette の syndrome measurement を実行した。QEC方向への前進も見られる。
結果として、異種間 entangling gate fidelity は 0.975 ± 0.002、QND syndrome measurement fidelity は 2-qubit で 0.933(12)、3-qubit で 0.865(17)。著者らはこれを、従来の Rb–Cs 結果に対する order-of-magnitude improvement と位置づけており、詳細シミュレーションでは実験改良により 0.997 超まで伸ばせる見通しも述べている。
さらに著者たちは明示的に、single-species では syndrome extraction にしばしば必要になるmeasurement zone への移動, shelving / hidingを、dual-species なら回避できると位置づけている。

---

## 3. 周辺・比較

### 3.1 実験: mixed-species / dual-isotope assembly

**Sheng et al., PRL 128, 083202 (2022)**
*Defect-Free Arbitrary-Geometry Assembly of Mixed-Species Atom Arrays*

こちらは 85Rb/87Rb の mixed-species assembly。mixed-species（正確には dual-isotope） 単一原子アレイを、任意形状・任意組成比で bottom-up に組み上げた初実証。二種原子 NAQC の中では、gate や readout ではなく array assembly / rearrangement 側の実験.
直接的には Rb/Cs の QND 路線ではないが、mixed-species atom-by-atom assembly が実際に可能であることを示した点で、dual-species hardware の一般論として重要。
初期のランダム配置から、heuristic heteronuclear algorithm (HHA) を用いて原子を並べ替え、2D の 6×4 mixed-species array を作成した。アルゴリズムは user-defined geometry と two-species atom number ratio の両方に対応している。
NAQCに対しては、data/ancilla を二種で分けた interleaved layout や checkerboard 配置のような、mixed-species アーキテクチャを実際に作るための配列生成技術の補助的な実験の立ち位置。

---

### 3.2 理論: dual-species quantum networking

**Young et al., Appl. Phys. B 128, 151 (2022)**
*An architecture for quantum networking of neutral atom processors*

二種原子中性原子計算機を量子ネットワークのノードとして使うためのアーキテクチャ提案で、一方の原子種を通信（atom–photon entanglement）用、もう一方を計算・記憶用に分けることを提案した。
由空間集光より cavity の方が有利で、特にcavity 内でそのまま冷却・捕獲できる near-concentric 設計により、原子輸送を避けつつ高レート化できると示した。
dual-species を単なる readout 分離ではなく、modular / distributed neutral-atom quantum computing に拡張した点が重要。通信光が data qubit に直接当たらないのでクロストークを抑えられ、通信原子をQND 測定や measurement-based error correction にも流用できることを整理しているか。

---

### 3.3 実験: single-species による mid-circuit measurement の比較対象

**Graham et al., PRX 13, 041051 (2023)**
*Mid-circuit measurements on a single species neutral alkali atom quantum processor*

単一種の中性アルカリ原子アレイで、mid-circuit measurement を実証した論文。dual-species を使わず、data qubit を保護状態へ shelving し、ancilla だけを非破壊測定する方式を示した。
具体的にはdata qubit を protected hyperfine-Zeeman sub-states に退避し、ancilla 測定中は microwave repumping で測定 fidelity を上げ、さらに dynamical decoupling で shelved data qubit の coherence を保ちながら、測定後に計算基底へ戻すプロトコルを実装した。
data qubit の量子状態は定数位相ずれを除いてよく保存され、SPAM-corrected process fidelity は 
97.0(5)%。ancilla 測定 fidelity は、状態準備誤差補正後に ∣0⟩ で 94.9(8)%、∣1⟩ で 95.3(1.1)% と報告している.
これは dual-species 論文ではないが、neutral-atom で mid-circuit measurement が本当に可能だと示した重要な基準点。dual-species の利点（測定クロストーク回避や保護操作の簡略化）を際立たせる比較対象となる。

---

### 3.4 実験: single-species Yb による measurement / refill

**Norcia et al., PRX 13, 041034 (2023)**
*Mid-circuit qubit measurement and rearrangement in a (^{171})Yb atomic array*

単一種の171Yb arrayで、他の qubit をできるだけ乱さずに一部の qubit だけを途中測定し、その結果に応じて ancilla サイトを補充するところまで示した。
具体的には,狭線幅遷移を使った imaging で、nondestructive・state-selective・site-selective detection を実現した。さらに、site-specific light shift で一部の原子を imaging 光から“隠す”ことで、選んだ qubit だけを測定できるようにした。加えて、測定結果に基づくconditional refilling と、MOT ロードが他の qubit のコヒーレンスを大きく壊さないことも示した。
mid-circuit measurement を行っても、測定しない残りの qubit への影響は percent-level に抑えられ、ancilla の再利用に向けた条件付き補充まで実証した。著者たちはこれを、continuous operation に向かう実証として位置づけている。
dual-speciece との比較対象となる。

---

### 3.5 理論: グローバル駆動だけのUNiversal Quantum Computing.

**Cesa & Pichler, PRL 131, 170601 (2023)**
*Universal Quantum Computation in Globally Driven Rydberg Atom Arrays*

dual-species Rydberg array を土台にして、静的な原子配置とグローバル共鳴パルス列だけで任意の量子回路を実行できると示した論文。局所制御を減らした neutral-atom QC のアーキテクチャ提案.
具体的には、2つの構成を与えている。1つ目は、量子回路そのものを原子のトラップ配置に埋め込む方式。2つ目は、原子配置は回路に依らず固定し、アルゴリズムをグローバル駆動系列に埋め込む方式。原子数の二次オーバーヘッドで局所制御を不要にした普遍量子計算機が実現できると主張している。
つまり dual-species は、QEC の readout hack に留まらず、control model 自体を変える資源にもなりうる。

---

### 3.6 実験: dual-isotope / hybrid Yb の low-crosstalk readout

**Nakamura et al., PRX 14, 041062 (2024)**
*Hybrid Atom Tweezer Array of Nuclear Spin and Optical Clock Qubits*

171Yb の nuclear-spin qubit を data、174Yb の optical clock qubit を ancilla/readout に使う hybrid atom array を実現し、低クロストークな ancilla 測定の基盤を示した論文。
具体的には、fermionic 171Yb を長コヒーレンスな data qubit、bosonic 174Yb を非破壊 readout 可能な ancilla qubitとして使う設計を実装し、174Yb の imaging light が 171Yb のコヒーレンスに与えるクロストークを評価した。
結果として、174Yb の 399 nm probe と 556 nm cooling を使う Hahn echo 測定で、20 ms の露光後でも99.1(1.8)% の coherence を保持し、imaging fidelity 0.9992、survival probability 0.988 を報告している。さらに 556 nm probe を使う Ramsey 測定では、コヒーレンスへの影響はほぼ無視できるとしている。
これは dual-species/dual-isotope の別系統で、発想は同じ。
- long-coherence data qubit
- easy-to-read ancilla qubit
- low-crosstalk measurement

Comment:この論文の主眼はhybrid array の実現と readout crosstalk の評価であって、Rydberg entangling gate、syndrome extraction、反復 QEC を実証したわけではない.dual-isotopeの方はどのくらい進んでいるのか？dual-speciesと同じ議論をどこまで適応していいのか？

---

### 3.7 実験: dual-species array の簡素化

**Fang et al., Sci. Adv. (2025)**
*Interleaved dual-species arrays of single atoms using a passive optical element and one trapping laser*

dual-species array の物理を前に進めたというより、実装をかなり簡素化した論文。明るい tweezer と暗い bottle-beam trap を交互配置し、Rb と Cs の動的分極率の符号が逆になる波長域を使って、Rb は bright site、Cs は dark site に自動的に分かれて入る設計を示した。dual-species hardware をより簡素にスケールさせる方向として重要。
dual-species を新しい競合プラットフォーム”ではなく,既存 NAQC に載せる実装しやすいサブアーキテクチャに寄せた.Rb は bright site、Cs は dark site にしか入らないので、dual-species 配列の再配置問題も少し楽になるか。

---

### 3.8 実験: single-species. shuttling ではなく optical switching で回路を動かす.

**Radnaev et al., PRX Quantum 6, 030334 (2025)**
*A universal neutral-atom quantum computer with individual optical addressing and non-destructive readout*

別光学アドレッシング＋非破壊 readout を組み合わせて、neutral-atom QC を「動かして普遍計算する装置」として示した論文。dual-species の論文ではなく、高速な単一プラットフォーム型 neutral-atom architecture の代表例. single-species 路線だが、重要な比較対象。
具体的には、従来の mid-circuit shuttling 依存ではなく、individual optical addressing で局所操作を実行し、さらに nondestructive readout を組み込んだ。これにより、ゲート速度の律速を原子移動ではなく光学スイッチング時間に移した。
結果は、CZ fidelity 99.35(4)%、local single-qubit R_Z gate fidelity 99.902(8)%、non-destructive readout の損失 0.9(3)% を報告している。さらに、atom-loss event を除外すると CZ fidelity 99.73(3)% を測定しており、erasure conversion と結びつけている。
ancilla 分離・低クロストーク readout・spectator/QEC 補助で効くサブアーキテクチャとして位置づけられるか。

重要なのは、「no-shuttling」には2種類あること:
- measurement zone への transport を避ける no-shuttling（dual-species が狙うもの）
- entangling gate のための中間 shuttling 自体を避ける no-shuttling（individual addressing 路線）

---

### 3.9 実験: single-species FT architecture の実験

**Bluvstein et al., Nature 649, 39–46 (2026)**
*A fault-tolerant neutral-atom architecture for universal quantum computation*

こちらは zoned / shuttling / reservoir を含む大規模 FTQC 路線の実験。storage, entangling, readout, reservoir zones を持ち、repeated QEC と entropy removal を含む包括的アーキテクチャを示した。
具体的にはsurface code を使って repeated QEC の効果を調べ、4 round の characterization circuit で 2.14(13) 倍の below-threshold performance を示した。さらに、transversal gates と lattice surgery による logical entanglement、3D \[\[15,1,3\]\] code を使った transversal teleportation による universal logical gate、そして mid-circuit qubit re-use による深い回路実行を実装した。
結果は、non-destructive, spin-resolved readout と mid-circuit reinitialization により、実験サイクル率を 2 桁改善し、\[\[7,1,3\]\] や \[\[16,6,4\]\] code を用いた dozens of logical qubits と hundreds of logical teleportations が可能な深回路プロトコルへの道を示した。

dual-species 路線を考える上では、これはsingle-species 側がどこまで進んだかを示す論文である。競合とは見なくてもいいのではないか。
-  Harvard路線: zoned separation / transport / reservoir を前提に early FTQC を組む？
- dual-species 路線: species separation により in-place ancilla measurement を作る. サブアーキテクチャとしての利用も考えられる。

---

## 4. dual-species の現状

### 長所
1. species-selective readout による low-crosstalk ancilla measurement
2. ancilla/data の役割分離が自然
3. continuous reload / replenishment と相性が良い
4. networking interface や optical ancilla との機能分離に拡張しやすい
5. in-place syndrome extraction を設計しやすい

###  短所・弱点
1. interspecies gate が単一種の gate fidelity にまだ追いついていないことが多い
2. trap / cooling / imaging / laser system が複雑になる
3. 異種原子間 blockade の最適化、クロストーク評価、エラーモデルが難しい
4. ancilla readout は楽になっても、reset / reload / decoder / real-time feedback は別問題
5. 大規模 surface code / LDPC 実装でのcorrelated error の実験的把握はこれから

---

## 5. 今後

1. ancilla-only repeated syndrome cycles の実証
2. reset / reload / feed-forward を含む dual-species QEC cycle の実証
3. larger plaquette / small patch での repeated stabilizer extraction
4. cross-species correlated error / leakage / measurement backaction の characterization
5. dual-species を zoned architecture / static layout / networking interface などの既存アーキテクチャとどう結びつけるか

---
