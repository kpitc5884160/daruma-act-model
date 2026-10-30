# CRANE+ V2 だるま落とし用 ACT モデル

CRANE+ V2を使った、Gazebo上のだるま落とし課題向けACTモデルです。

## 状態と制約

本モデルはカメラ画像を使わず、関節状態を入力とする模倣学習モデルです。

Gazebo上で動作を確認しましたが、だるま落としを安定して成功させる段階には至っていません。  
（参考用デモ動画：https://youtu.be/Ck91Yuogq34）

## このリポジトリに含まれるもの

- `artifacts/daruma_act_model/config.json`：ACTモデル設定
- `artifacts/daruma_act_model/model.safetensors`：学習済み重み（Git LFS）

## 動作確認環境

- Windows 11 / Docker Desktop
- Ubuntu 22.04
- ROS 2 Humble
- Gazebo Classic 11 / Ignition Gazebo 6
- CRANE+ V2
- LeRobot ACT

Docker環境のベースには、[AI-Robot-Book-HumbleのDocker環境](https://github.com/AI-Robot-Book-Humble/docker-ros2-desktop-ai-robot-book-humble)を使用しました。

## 依存パッケージ

### CRANE+ V2とGazebo

`~/airobot_ws`に、CRANE+ V2のROS 2/Gazeboパッケージが必要です。

この課題では、ハンマーを取り付けたロボットモデルと、だるま落とし用ワールドを追加しています。

### physicalai_pubsub（講義のPub/Subサンプル）

このパッケージは、CRANE+ V2の関節状態購読や動作指令の例を試すために使用します。ACTモデルの推論ノードではありません。

```bash
mkdir -p ~/pai_ws/src
cd ~/pai_ws/src
git clone https://github.com/matsuolab/physicalai_pubsub.git

cd ~/pai_ws
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install --packages-select physicalai_pubsub
source install/setup.bash
```

Pub/Subのサンプルは、次のコマンドで実行できます。

```bash
ros2 run physicalai_pubsub state_subscriber
ros2 run physicalai_pubsub action_publisher
```

### LeRobotと推論コード

推論には、別途`pai_training`プロジェクトと、その中の` scrips/infer.py `が必要です。この推論コードは本リポジトリには含まれていません。

## モデルの取得

Git LFSをインストールした環境で、次のコマンドを実行します。

```bash
git lfs install
git clone https://github.com/kpitc5884160/daruma-act-model.git
cd daruma-act-model
git lfs pull
```

## Gazeboでの実行

カスタムワールドとCRANE+ V2モデルを含む`~/airobot_ws`がセットアップ済みであることを確認し、Gazeboを起動します。

```bash
source /opt/ros/humble/setup.bash
source ~/airobot_ws/install/setup.bash
ros2 launch crane_plus_gazebo daruma_with_table.launch.py
```

別のターミナルでコントローラの状態を確認します。

```bash
source /opt/ros/humble/setup.bash
source ~/airobot_ws/install/setup.bash
ros2 control list_controllers
```

各コントローラが`active`になったら、推論コードを実行します。

```bash
source /opt/ros/humble/setup.bash
source ~/airobot_ws/install/setup.bash
cd ~/pai_training

pixi run python scripts/infer.py \
  --model-path ~/daruma-act-model/artifacts/daruma_act_model
```

## ライセンスと出典

`physicalai_pubsub`およびDocker環境のライセンス・利用条件は、それぞれの元リポジトリを参照してください。

- [physicalai_pubsub](https://github.com/matsuolab/physicalai_pubsub)
- [AI-Robot-Book-Humble Docker環境](https://github.com/AI-Robot-Book-Humble/docker-ros2-desktop-ai-robot-book-humble)
