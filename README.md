# SolenoidStokeUNIT

<img src="https://github.com/akita11/SolenoidStrokeUNIT/blob/main/SolenoidStrokeMeasureControl_UNIT.jpg" width="240px">

ソレノイドのストローク位置によってインダクタンスが変化し、PWM制御時の電流波形が変化することを利用して、ソレノイドのPWM制御とストローク位置の計測（推定）を同時に行うための回路です。

**【注意：重要】**
実際の制御・計測の動作には、ソレノイドごとに特性の計測を行う必要があり、また特性の個体差、温度特性なども考慮する必要があります。そのため、**「これを使ってサンプルプログラムを書き込めば間違いなく動作する（制御ができる）」というものではなく、あくまでもこの原理検証の実験のための回路**ということをご理解ください。測定原理やサンプルプログラムは[こちら](https://github.com/akita11/SolenoidStrokeMeasureControl)に適宜まとめて更新していきます。


## 使い方

オレンジ色のコネクタ(VH3.96-4p)に、ソレノイド用の電源とソレノイドを接続します。M5Stackの[BAVGソケット](https://www.switch-science.com/products/7234)を使うと、電源は2.1mmプラグ、負荷はスプリング端子で接続できて便利です。

Grove端子の3番("PWM"と表記）にPWM波形を加えてソレノイドをPWM駆動し、その駆動電流が電圧波形としてGrove端子の4番（"ADC"と表記）に出力されます。

なおArduinoやM5Stack (Core2/CoreS3)用のサンプルプログラムは[こちら](https://github.com/akita11/SolenoidStrokeMeasureControl)にあります（あくまでも参考用のものです）。Measure_...がパラメータ計測用のプログラム、Control...が位置計測（推定）・位置制御用のプログラムです。（ESP32用はM5Stack Core2のPortAに本ボードを接続して使用します）

### 増幅率の調整

ソレノイドの駆動電流を0.2Ωの抵抗で電圧変換し、それをオペアンプ（非反転増幅回路）で増幅して"ADC"端子に出力しています。その増幅率は、用いるソレノイド・電源電圧からきまる駆動電流にあわせて、適切な範囲に調整する必要があります。"ADC"端子の波形をオシロ等で計測し、用いるマイコンのADCの入力電圧範囲に収まるように増幅率を調整します。増幅率は基板裏面のハンダジャンパJP3/JP4JP5で調整できます。（JP3は初期状態でONになっていますので、OFFにする場合はパターンをカットしてください）

- JP3/JP4/JP5=ON/OFF/OFF : 101倍
- JP3/JP4/JP5=ON/ON/OFF : 39倍
- JP3/JP4/JP5=ON/OFF/ON : 20倍
- JP3/JP4/JP5=ON/ON/ON : 16倍
- JP3/JP4/JP5=OFF/OFF/OFF : R1にとりつける抵抗から決まる倍率（(R1÷1[kΩ]+1)倍）

<img src="https://github.com/akita11/SolenoidStrokeUNIT/blob/main/SolenoidStrokeMeasureControl_UNIT_b.jpg" width="240px">


## 測定原理

ソレノイドは電気回路的にはインダクタで、そのインダクタンスはストローク（プランジャ）位置で変化します。そのためON時の充電電流の波形は、ストローク位置で変化します。ただし充電開始時の電流はOFF期間中の放電特性できまるため、ゼロからとは限りません。またPWMのON時間（PWMのデューティー比）によっても波形は変化します。

ソレノイドによっては、ON後の2箇所の時点（例えば100us後と700us後）での電流の差が、PWMのON時間（PWMのデューティー比）ごとにストローク位置に伴って変化するため、与えているPWM ON時間とその電流差から、ストローク位置を計測（推定）することができます。現時点で試した範囲では、TakahaのCBS0730シリーズはこの特性をもっています。（詳細は[arXivの論文](https://arxiv.org/abs/2405.11721)にまとめてあります）

ただしソレノイドによっては、この電流差がストロークによってほとんど変化しないため、この方法では位置を計測することができません。そのため、複数点の電流の計測値から、予め計測した位置と電流値との差（の総和）が最小となるように反復計算で位置を求めるアルゴリズムを開発中です。まだ安定性や精度の検証は不十分ですが、参考用に上記GitHubにArduinoとESP32用のソースコードをおいてあります（計測データはTakahaのLongStrokeSolenoidのもの）。



## Author

Junichi Akita (akita@ifdl.jp, @akita11)




