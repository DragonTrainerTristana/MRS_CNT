<h1 align="center">A Lightweight Game-Engine Digital Twin for Conductivity in CNT/PDMS Composites</h1>

<p align="center">
  Static conductivity, a second silica filler, and the piezoresistive response under load — handled by one simulator on one workstation.
</p>

<p align="center">
  <b>Seung Hyun Oh</b>, Seunggwan Oh, Sang Hyun Lee &nbsp;·&nbsp; Korea University &nbsp;·&nbsp; MRS Fall 2026, Symposium NM04
</p>

<p align="center">
  <code>Unity DOTS / ECS</code> &nbsp; <code>Burst</code> &nbsp; <code>GPU compute (HLSL)</code> &nbsp; <code>C#</code>
</p>

---

We model how a carbon-nanotube (CNT) network conducts inside a PDMS rubber, and we run the whole thing on top of the Unity game engine. A game engine already places and updates millions of moving objects on an ordinary PC, so one workstation can cover cases that usually take separate codes or a cluster: the static conductivity, a second non-conducting silica filler, and the change in resistance when the material is stretched, compressed, or bent. This page collects the figures and runs behind our MRS Fall 2026 abstract.

> PDMS 고무 안에서 탄소나노튜브(CNT) 네트워크가 전기를 어떻게 흘리는지를 시뮬레이션하고, 이 전부를 Unity 게임 엔진 위에서 돌린다. 게임 엔진은 이미 수백만 개의 움직이는 물체를 일반 PC에서 배치하고 갱신하므로, 보통 별도 코드나 클러스터가 필요한 경우들 — 정적 전도도, 비전도성 실리카 2차 필러, 그리고 늘리거나 누르거나 굽혔을 때의 저항 변화 — 을 워크스테이션 한 대로 다룬다. 이 페이지는 2026 MRS Fall 초록을 뒷받침하는 그림과 실험 결과를 모아 둔 것이다.

---

## 1. How the simulator is put together

<p align="center">
  <img src="Images/Figure_1_Architecture_MRS.png" width="900" alt="Architecture of the simulator, from inputs through the engine to the conductivity and piezoresistive outputs">
</p>

The path from input to output. You fix a handful of numbers — tube diameter and aspect ratio, the contact resistance, the tunnelling decay, the loading in wt%, the box size, a random seed. The model drops straight, mutually penetrable rods into the box at random positions and directions (random sequential adsorption), keeps every rod as a flat data record that all CPU cores update at once, and finds the tube-to-tube contacts on a spatial grid using the GPU. Each contact becomes a tunnelling resistor, `R = R_c · exp(κ·d)`, where `d` is the surface gap. Solving that resistor network gives the conductance `G`, and the box geometry turns it into a conductivity `σ`. Two modules branch off the same network: a silica filler that crowds the tubes, and a stretch-and-bend mode that moves them and re-solves the circuit. The outputs are the `σ`–wt% percolation curve and the piezoresistive `R/R₀` response.

> 입력에서 출력까지의 흐름이다. 사용자는 몇 개의 값만 정한다 — 튜브 지름과 가로세로비(AR), 접촉 저항, 터널링 감쇠, 농도(wt%), 상자 크기, 난수 시드. 모델은 곧은 막대를 서로 통과 가능한 상태로 상자 안 임의 위치·방향에 떨어뜨리고(RSA), 막대 하나하나를 평평한 데이터 레코드로 두어 모든 CPU 코어가 동시에 갱신하며, 튜브 간 접촉을 GPU 위 공간 격자에서 찾는다. 접촉 하나하나는 터널링 저항 `R = R_c · exp(κ·d)`(여기서 `d`는 표면 간 간격)이 된다. 이 저항망을 풀면 전도도 `G`가 나오고, 상자 형상을 통해 전도율 `σ`로 바뀐다. 같은 네트워크에서 두 모듈이 갈라진다 — 튜브를 밀어내는 실리카 필러, 그리고 튜브를 움직이고 회로를 다시 푸는 인장·굽힘 모드. 출력은 `σ`–wt% 퍼콜레이션 곡선과 압저항 `R/R₀` 응답이다.

---

## 2. The control panel

<p align="center">
  <img src="Images/Figure_2_Interface.jpeg" width="760" alt="Unity Game-view control panel for the conductivity measurement">
</p>

The settings screen, captured straight from the Unity Game view. You choose the CNT type (here a 6 µm "Long" tube), turn the silica filler on or off and pick its size (5 µm or 25 nm), set how many repetitions to average, and name the output CSV. "Start Experiment" launches the wt% sweep. The translucent box on the right is the simulation volume that fills up as the run proceeds.

> Unity Game 뷰에서 그대로 캡처한 설정 화면이다. CNT 종류(여기서는 6 µm "Long" 튜브)를 고르고, 실리카 필러를 켜거나 끄고 그 크기(5 µm 또는 25 nm)를 선택하며, 평균 낼 반복 횟수를 정하고, 출력 CSV 이름을 적는다. "Start Experiment"가 wt% 스윕을 시작한다. 오른쪽 반투명 상자가 실행 중 채워지는 시뮬레이션 부피다.

---

## 3. A sweep, recorded live

<p align="center">
  <img src="Images/Figure_3_MRS_Poster.gif" width="860" alt="Live screen capture of a weight-fraction sweep building the conductivity curve">
</p>

A single run, recorded as it happens. As the loading climbs from 0.05 to 2 wt%, the box fills with tubes and the conductivity curve on the right is drawn one point at a time — linear on top, semi-log below. Each point is one fully solved network, and the run shown steps through twenty-three concentrations.

> 실행 장면을 그대로 녹화한 것이다. 농도가 0.05에서 2 wt%로 올라가는 동안 상자가 튜브로 채워지고, 오른쪽 전도도 곡선이 한 점씩 그려진다 — 위는 선형, 아래는 반로그. 각 점은 완전히 풀린 네트워크 하나이며, 이 실행은 농도 스물세 단계를 거친다.

---

## 4. What the network looks like

The same box, before and after percolation. Near the threshold only a few tubes reach across the volume and most branches are dead ends; at higher loading the network is dense and many separate paths carry current at once. Adding a silica filler (the red volume) takes up room and pushes the tubes into the space that is left, which bends and reroutes the conducting paths.

> 같은 상자를 퍼콜레이션 전후로 본 것이다. 임계점 근처에서는 부피를 가로지르는 튜브가 몇 개뿐이고 대부분의 가지가 막다른 길이지만, 함량이 높아지면 네트워크가 조밀해지고 여러 독립 경로가 동시에 전류를 나른다. 실리카 필러(빨간 부피)를 넣으면 그 불활성 부피가 자리를 차지해 튜브를 남은 공간으로 밀어내고, 그 결과 전도 경로가 휘고 다시 짜인다.

<p align="center">
  <img src="Images/Figure_5_Network_Gallery.png" width="900" alt="Five CNT network snapshots: two without silica (sparse, dense) and three with silica (sparse, intermediate, dense)">
</p>

---

## 5. Static conductivity

<p align="center">
  <img src="Images/Figure_4_Static_Results.png" width="980" alt="Conductivity versus loading for four aspect ratios, shown linear, semi-log, and as a percolation fit">
</p>

Conductivity against loading for four aspect ratios (50, 100, 200, 400), shown three ways: linear, semi-log, and a log–log percolation fit. The three electrical inputs are fixed across the whole set. The threshold moves from about 0.4 wt% at AR 50 down to about 0.05 wt% at AR 400 — the `1/AR` drop expected for slender fillers, and the reason a long tube needs so little material to start conducting. The fitted slope `t` sits near 2, the three-dimensional percolation value. We read the shape of the rise and this scaling from the figure; the contact resistance is what sets the absolute level.

> 가로세로비 네 종류(50, 100, 200, 400)에 대한 농도-전도도 관계를 세 방식 — 선형, 반로그, 로그-로그 퍼콜레이션 피팅 — 으로 보인다. 세 전기 입력값은 전체에 걸쳐 고정한다. 임계점은 AR 50의 약 0.4 wt%에서 AR 400의 약 0.05 wt%로 내려가며, 이는 가늘고 긴 필러에서 기대되는 `1/AR` 감소이자 긴 튜브가 아주 적은 양으로도 전도를 시작하는 이유다. 피팅 기울기 `t`는 3차원 퍼콜레이션 값인 2 근처에 있다. 이 그림에서는 상승의 모양과 이 스케일링을 읽으며, 절대 크기는 접촉 저항이 정한다.

---

## 6. How filler size moves the result

The silica modules let us change one thing at a time — the size of the second filler — and watch where the conductivity goes.

> 실리카 모듈로는 한 번에 하나 — 2차 필러의 크기 — 만 바꿔 가며 전도도가 어디로 가는지 본다.

<p align="center">
  <img src="Images/Fig3.png" width="520" alt="Resistivity versus CNT loading, with and without 30 wt% silica of two sizes">
  &nbsp;&nbsp;
  <img src="Images/Fig4.png" width="520" alt="Resistivity versus silica content at fixed 1 wt% CNT">
</p>

On the left, resistivity against CNT loading with and without 30 wt% silica. Big silica (5 µm) lowers the resistivity a little, while the same amount of small silica (25 nm) raises it: the finer powder spreads over far more surface and gets between the tubes. On the right, resistivity against silica content at a fixed 1 wt% CNT. The two sizes pull in opposite directions, and both start from the same no-silica baseline near 10 S/m.

> 왼쪽은 30 wt% 실리카가 있을 때와 없을 때의 CNT 농도-비저항 관계다. 큰 실리카(5 µm)는 비저항을 약간 낮추는 반면, 같은 양의 작은 실리카(25 nm)는 높인다. 더 고운 분말이 훨씬 넓은 표면에 퍼져 튜브 사이를 가로막기 때문이다. 오른쪽은 CNT를 1 wt%로 고정한 채 실리카 함량을 바꾼 것이다. 두 크기가 반대 방향으로 작용하며, 둘 다 실리카 없는 기준선(약 10 S/m)에서 출발한다.

---

## 7. Why the network stays selective

<p align="center">
  <img src="Images/distant_distribution_1wt.jpeg" width="820" alt="Histogram of surface-to-surface gaps between tube pairs at 1 wt%">
</p>

How close the tubes actually sit to one another. The histogram is the surface-to-surface gap between every tube pair at 1 wt% (AR 400). Only the thin sliver to the left of the 2 nm cutoff carries any tunnelling current; the great majority of pairs are microns apart and contribute nothing. A small contact resistance and a steep tunnelling decay therefore still leave a sparse, selective set of conducting junctions.

> 튜브들이 실제로 서로 얼마나 가까이 있는지 보여준다. 히스토그램은 1 wt%(AR 400)에서 모든 튜브 쌍의 표면 간 간격이다. 2 nm cutoff 왼쪽의 얇은 부분만 터널링 전류를 나르고, 대다수 쌍은 수 마이크론씩 떨어져 아무 기여도 하지 않는다. 따라서 작은 접촉 저항과 가파른 터널링 감쇠가 있어도 전도 접합은 희박하고 선택적으로 남는다.

---

## What to read here, and what not to

The model takes its three electrical inputs — junction contact resistance, the tunnelling term, and the rods' own resistance — from published ranges, and holds them fixed across every run. The curves therefore show trends, scaling, and the shape of the transition, with the junction contact resistance setting the overall level, rather than a conductivity calibrated to one experiment. The rods are penetrable (soft-core), so overlaps count as van der Waals contacts instead of shorts, and the network reaches real sample loadings without the jamming that stops hard-core models.

> 모델은 세 전기 입력값 — 접합 접촉 저항, 터널링 항, 막대 자체 저항 — 을 문헌 범위에서 가져와 모든 실행에 걸쳐 고정한다. 따라서 곡선은 특정 실험에 맞춘 절대 전도도가 아니라 추세·스케일링·전이의 모양을 보여 주며, 전체 크기는 접합 접촉 저항이 정한다. 막대는 서로 통과 가능한(soft-core) 모델이라 겹침은 단락이 아니라 반데르발스 접촉으로 세며, hard-core 모델을 멈춰 세우는 잼(jamming) 없이 실제 시료 수준의 농도까지 도달한다.

---

<p align="center">
  Figures are posted here for inspection. The simulator code is available on request.<br>
  <sub>MRS Fall 2026 · Symposium NM04 · Korea University</sub>
</p>
