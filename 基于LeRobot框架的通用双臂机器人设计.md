---
    						 Powered  By yuki
---

# 基于 LeRobot 框架 的 通用双臂机器人设计



参考：

[lerobot 官方 github 仓库]:https://github.com/huggingface/lerobot/tree/main:

# 项目简介

本研究设计并实现一种基于 LeRobot 框架的通用双臂机器人，这是一个可远程遥操的系统，用于模仿双手操作，通过对不同任务的少量数据采集并训练，能够较好地执行各种工业上以及生活上的任务，通用性高。



# 一. 环境配置

------------------------------------

## 1. 从 github 下载源码

***下载慢记得挂 vpn！***

```shell
git clone https://github.com/huggingface/lerobot.git
cd lerobot
```

***还是不行的话直接在网页下载解压…***

## 2. 新建环境

```shell
conda create -y -n lerobot python=3.10
conda activate lerobot
```

## 3. 安装 pytorch 

> 在官网根据自己版本选 https://pytorch.org/

```shell
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
```

## 4. 安装 lerobot

```
pip install -e .
```



# 二. 硬件组装

----------------------------------------------

## 1. 购买硬件

-  [双臂lerobot硬件清单.xlsx](双臂lerobot硬件清单.xlsx) 


## 2. 安装飞特舵机驱动库

（每次都记得先激活lerobot环境）

```shell
pip install -e ".[feetech]"
```

## 3. 查找主从臂对应的端口号

```shell
python lerobot/scripts/find_motors_bus_port.py
```

## 4. 给予对应端口权限

```shell
sudo chmod 666 /dev/ttyACM*
```

## 5. 更新配置文件

- [x] 找到 `lerobot/lerobot/common/robot_devices/robots/configs.py` 文件

- 找到`so100`相关的配置代码块，将 第432行 和 第449行 修改成对应的主从臂端口号

## 6. 依次初始化各舵机

```shell
python lerobot/scripts/configure_motor.py \
  --port /dev/ttyACM2 \
  --brand feetech \
  --model sts3215 \
  --baudrate 1000000 \
  --ID 1
```

*将 第2行的端口号改成对应的，最后一行的ID号改成1～6，初始化完毕后保持舵机齿轮位置不动！*

**！安装主臂时，初始化舵机后需要拆掉最顶上的齿轮使得舵机失力！**

## 7. 安装机械臂并固定

安装教程：

- [从臂组装步骤]: https://www.bilibili.com/video/BV1GJmRYvENG?spm_id_from=333.788.videopod.sections&vd_source=0a14d99fc872e26e192d2ebfe108ae2b

- [主臂组装步骤]:https://www.bilibili.com/video/BV1dZmoYjEFW?spm_id_from=333.788.videopod.sections&vd_source=0a14d99fc872e26e192d2ebfe108ae2b

Tips：记得理好线打好标签！

**安装一定要对准每一个关节位置，不然遥操时会很奇怪！！！**

## 8. 安装摄像头

- [x] 安装**全局摄像头**（手机支架固定，只拍从臂工作空间）
- [x] 安装**左臂腕部摄像头** （可使用纳米胶和挣扎固定在腕部，使用橡皮擦垫高，画面中要拍到爪子）
- [x] 安装**右臂腕部摄像头**

## 9. [遥操出问题再看]

*<font color="red"><u>**问题：遥操不同步（主从臂位置，旋转方向不同步）**</u></font>*

**解决方案：**

<u>***若不慎齿轮没对准，可以使用 `飞特调试软件FD` 重新进行中位设置。***</u>

（其实直接用软件设置比前面步骤要更简单…）

**FD进行中位校准STEP：**

1. 端口号选对应，波特率选最大，打开串口
2. 搜索舵机，修改对应的ID号
3. 将机械臂摆放至中位，选中每一个舵机点击 `中位校准`
4. `中位校准` 完后，删除原标定文件夹，重新标定



# 三. 硬件校准与遥操测试

------------------------------------------------

## 1. 修改主从臂配置文件端口号

详见 `二. 硬件组装` 部分

```shell
python lerobot/scripts/find_motors_bus_port.py
```

```shell
ls /dev/ttyACM*
```

```shell
sudo chmod 666 /dev/ttyACM*
```

## 2. 从主臂初始位姿标定

```shell
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --robot.cameras='{}' \
  --control.type=calibrate \
  --control.arms='["main_follower"]'
```

```shell
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --robot.cameras='{}' \
  --control.type=calibrate \
  --control.arms='["main_leader"]'
```

根据官网图片依次标定好从主臂各3个位姿。

## 3. [校准报错再看]

*<font color="red"><u>**问题：报错no status packet**</u></font>*

**解决方案：**

基本属于硬件问题，主要检查接线。

确实没找出问题可以用飞特调试软件调试，检查每一个舵机和接线哪里有问题。

（我当时因为一个舵机ID设错了卡了很久…笨拙诅咒…）

## 4. 遥操测试（不带摄像头）

```shell
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --robot.cameras='{}' \
  --control.type=teleoperate
```

## 5. 查看对应相机ID号

```shell
python lerobot/common/robot_devices/cameras/opencv.py \
    --images-dir outputs/images_test
```

## 6. 修改相机配置文件

- [x] 找到 `lerobot/lerobot/common/robot_devices/robots/configs.py` 文件
- 找到`so100`相关的配置代码块，将 第466行 和 第472行 修改成对应的相机ID

## 7. 修改`PID`控制器配置文件

- [x] 打开 `lerobot/common/robot_devices/robots/manipulator.py` 文件
- 找到`PID`参数的配置代码块，将 第422行 的 `I` 参数值修改成 0 （默认为32）

> 此操作会使得机械臂运动时不再那么抖，`PID` 参数值都可以自行调试修改

## 8. 遥操测试（带摄像头）

```shell
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --control.type=teleoperate
```

## 9. [带摄像头遥操卡住 再看]

<font color="red">***<u>问题：运行 带摄像头的遥操 代码时，摄像头画面出不来，卡在那里无法遥操</u>*** </font>

**解决方案：**

1. 重装opencv：

    ```shell
    conda install -y -c conda-forge ffmpeg
    pip uninstall -y opencv-python
    conda install -y -c conda-forge "opencv>=4.10.0"
    ```

2. 安装gtk3：

    ```shell
    conda install -c conda-forge gtk3
    ```

3. 再次测试带摄像头的遥操测试，若出现libtiff库链接问题再用下面的解决方案，否则忽略。

<u><font color="red">***问题：ImportError: /home/yuki/miniconda3/envs/lerobot/lib/python3.10/site-packages/cv2/python-3.10/../../../.././libtiff.so.6: <br> 	   undefined symbol: jpeg12_write_raw_data, version LIBJPEG_8.0***</font></u>

**解决方案：**

1. 进入 `/home/yuki/miniconda3/envs/lerobot/lib` 文件夹

2. 找到并删除名为 `libtiff.so.6` 的软链接（其链接到的是`libtiff.so.6.1.0`）

3. 将更低版本的 `libtiff.so.5.2.0` 拖入当前文件夹

4. 创建名为 `libtiff.so.6` 的软链接，并链接到 `libtiff.so.5.2.0` 

    ```shell
    ln -s libtiff.so.5.2.0 libtiff.so.6
    ```

5. 重新测试带摄像头的遥操

## 10. 修改成3个摄像头

- [x] 找到 `lerobot/lerobot/common/robot_devices/robots/configs.py` 文件

- 找到`so100`相关的配置代码块，将从 第463 开始的camera部分代码修改成下面这段（多一个摄像头）：

    （记得修改对应的相机ID， “top” 等标签可以自行命名）

    ```python
        cameras: dict[str, CameraConfig] = field(
            default_factory=lambda: {
                "top": OpenCVCameraConfig(
                    camera_index=0,
                    fps=30,
                    width=640,
                    height=480,
                ),
                "white": OpenCVCameraConfig(
                    camera_index=2,
                    fps=30,
                    width=640,
                    height=480,
                ),
                "blue": OpenCVCameraConfig(
                    camera_index=4,
                    fps=30,
                    width=640,
                    height=480,
                ),
            }
        )
    ```

## 11. [摄像头出问题再看]

<font color="red">***<u>问题：运行 带摄像头的遥操 代码时，报 Timed out 错误，或者摄像头画面卡住</u>*** </font>

**解决方案：**

此问题考虑为 USB 上行带宽不够，注意每一个摄像头单独占一个电脑 USB 口（不要连在同一个扩展坞上）

或者尝试将等待时间设长一点避免超时。

- [x] 打开 `lerobot/common/robot_devices/cameras/opencv.py` 文件

- 找到`TimeoutError`的判断逻辑，将 第416行 数字 2 改成 20 甚至 50

> 我的电脑更神奇，分开接两个扩展坞反而打不开，接同一个扩展坞反而可以，不愧是bugUI…
>
> top 和 white 接雷电接口，blue 接另一个 USB口（这里又卡了好久…）

## 12. 双臂遥操测试（带摄像头）

- [x] **获取双臂标定文件夹**
    1. 进入 `lerobot/.cache/calibration` 文件夹（隐藏文件）下，新建命名为 `so100_bi` 文件夹
    2. 标定 blue 从主臂，将标定文件放入 `so100_bi` 文件夹，并修改标定文件名称 `blue_follower/leader.json`
    3. 标定 white 从主臂，将标定文件放入 `so100_bi` 文件夹，并修改标定文件名称 `white_follower/leader.json`

- [x] **修改配置文件**

    1. 打开 `lerobot/lerobot/common/robot_devices/robots/configs.py` 文件

    2. 第 423 行 标定文件路径修改为 `".cache/calibration/so100_bi"`

    3. 将机械臂部分配置代码修改成下面代码，对应修改端口号和命名

        ```python
        leader_arms: dict[str, MotorsBusConfig] = field(
                default_factory=lambda: {
                    "blue": FeetechMotorsBusConfig(
                        port="/dev/ttyACM2",
                        motors={
                            # name: (index, model)
                            "shoulder_pan": [1, "sts3215"],
                            "shoulder_lift": [2, "sts3215"],
                            "elbow_flex": [3, "sts3215"],
                            "wrist_flex": [4, "sts3215"],
                            "wrist_roll": [5, "sts3215"],
                            "gripper": [6, "sts3215"],
                        },
                    ),
                    "white": FeetechMotorsBusConfig(
                        port="/dev/ttyACM3",
                        motors={
                            # name: (index, model)
                            "shoulder_pan": [1, "sts3215"],
                            "shoulder_lift": [2, "sts3215"],
                            "elbow_flex": [3, "sts3215"],
                            "wrist_flex": [4, "sts3215"],
                            "wrist_roll": [5, "sts3215"],
                            "gripper": [6, "sts3215"],
                        },
                    ),
                }
            )
        
            follower_arms: dict[str, MotorsBusConfig] = field(
                default_factory=lambda: {
                    "blue": FeetechMotorsBusConfig(
                        port="/dev/ttyACM0",
                        motors={
                            # name: (index, model)
                            "shoulder_pan": [1, "sts3215"],
                            "shoulder_lift": [2, "sts3215"],
                            "elbow_flex": [3, "sts3215"],
                            "wrist_flex": [4, "sts3215"],
                            "wrist_roll": [5, "sts3215"],
                            "gripper": [6, "sts3215"],
                        },
                    ),
                    "white": FeetechMotorsBusConfig(
                        port="/dev/ttyACM1",
                        motors={
                            # name: (index, model)
                            "shoulder_pan": [1, "sts3215"],
                            "shoulder_lift": [2, "sts3215"],
                            "elbow_flex": [3, "sts3215"],
                            "wrist_flex": [4, "sts3215"],
                            "wrist_roll": [5, "sts3215"],
                            "gripper": [6, "sts3215"],
                        },
                    ),
                }
            )
        ```

- [x] **双臂遥操测试**

    ```shell
    python lerobot/scripts/control_robot.py \
      --robot.type=so100 \
      --control.type=teleoperate
    ```

    

# 四. 数据采集

----

## 1. 取消数据集上传

- [x] 打开 `lerobot/common/datasets/lerobot_dataset.py` 文件
    - 搜索 `pull_from_repo` 函数定义
    - 在 `None ：` 下一行添加 `return`  *（有两个这个函数）*

## 2. 设置环境变量

```shell
HF_USER=yuukiii
```

## 3. 录制数据集

```shell
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --control.type=record \
  --control.fps=30 \
  --control.single_task="lerobot double arm test03." \
  --control.repo_id=${HF_USER}/so100_test03 \
  --control.tags='["so100","tutorial"]' \
  --control.warmup_time_s=10 \
  --control.episode_time_s=45 \
  --control.reset_time_s=15 \
  --control.num_episodes=50 \
  --control.push_to_hub=false
```

- **control.single_task** ：任务描述，出现在 `Hugging Face` 数据仓库
- **control.episode_time_s**：每一集的最长录制时间，可按 `→` 提前结束当前阶段
- **control.reset_time_s**：每一集结束后的等待时间，此时间内需要将所有东西复位
- **control.num_episodes**：录制的集数
- **control.repo_id**：录制任务文件夹名称，会与后续数据可视化、模型训练、模型验证中参数相对应

以上参数可按需求自行修改。

**快捷操作：**

- `→` ：提前结束录制该集
- `←` ：重新录制该集
- `Esc` ： 退出录制

若中断后未录制完，可以添加下面参数继续录制。

<u>***继续录制数据集：***</u>

```shell
  --control.resume=true
```

## 4. 数据集可视化

> Tips：任务文件夹名称记得对应修改

- **尝试重新播放第一集数据：**

    ```shell
    python lerobot/scripts/control_robot.py \
      --robot.type=so100 \
      --control.type=replay \
      --control.fps=30 \
      --control.repo_id=${HF_USER}/so100_test \
      --control.episode=0
    ```

- **通过 `Hugging Face` 可视化全部数据集：**

    1. 运行数据可视化代码：

        ```shell
        python lerobot/scripts/visualize_dataset_html.py \
          --repo-id ${HF_USER}/so100_test03 \
          --local-files-only 1
        ```

    2. 打开本地服务器网站即可



# 五. 模型训练（旧版）

---

## 1. 修改训练参数

- [x] 打开 `lerobot/configs/train.py` 文件

- 第 26 行 `steps` 为训练总步数，默认10w步
- 第 114 行 `save_freq` 为保存模型的频率，默认每2w步保存一次
- 其它相关参数亦可根据需求自行修改

## 2. 远程连接实验室电脑

**`ToDesk` 远程连接**

- 设备代码：580 770 479
- 密码：Jnu@123456
- 系统密码：jnu

**Tips：**记得调高性能模式，会流畅不少！

## 3. 激活环境

```shell
conda activate lerobot
cd lerobot/
```

## 4. 登陆 `Hugging Face`

```shell
huggingface-cli login --token ${HF_TOKEN} --add-to-git-credential
```

```shell
git config --global credential.helper store
```

```shell
HF_USER=$(huggingface-cli whoami | head -n 1)
echo $HF_USER
```

## 5. `wandb` 可视化

1. 注册并创建你的 `API key`

    [wandb]:https://docs.wandb.ai/quickstart/

    > yuki26：1c8e6bbb9d8c05f76604431d49f0d4a3920d7d9e

2. 重登陆你的账号

    ```shell
    wandb login --relogin
    ```

3. 在官网可视化你的训练数据

    [wandb home]:https://wandb.ai/home

## 6. 模型训练

**查看显卡利用率：**

```shell
nvtop
```

**开始训练：**

```shell
python lerobot/scripts/train.py \
  --dataset.repo_id=${HF_USER}/so100_test \
  --policy.type=act \
  --output_dir=outputs/train/act_so100_test \
  --job_name=act_so100_test \
  --device=cuda \
  --wandb.enable=true
```

**Tips：**

- 开始训练会自动从 `Hugging Face` 下载数据，需要开`vpn`，下载完数据要立马关掉`vpn`，否则`ToDesk`会掉线
- 训练一段时间后会自动打开`wandb` 可视化工具
- 训练文件夹名称记得对应修改

## 7. [`ToDesk`掉线再看] 

<font color="red">***<u>问题：因连接 vpn 导致 ToDesk 掉线，无法远程连接</u>*** </font>

**解决方案：**

1. **终端连接实验室电脑**

    ```shell
    ssh star@nmg.frp.one -p 16440
    ```

    按照提示依次输入`yes`和`jnu`（密码）进行登录

2. **强制关闭 clash 进程**

    - 查询进程`ID`

        ```shell
        ps aux | grep clash
        ```

    - 杀死所有相关服务（上面查到的全部）（最后一个参数替换成相应`ID`）

        ```shell
        sudo kill -9 2155419
        ```

3. **重启 `ToDesk` 服务**

    ```shell
    todesk
    sudo systemctl restart todeskd.service
    ```

4. **重新连接`ToDesk` 或者使用终端继续完成操作**



# 六. 模型训练（新版）

- **新版模型训练使用的是 A100 服务器，采用本地数据集的方案**

## 1. 下载`FileZilla`

```shell
sudo apt install filezilla 
```

## 2. `Filezilla` 新建`A100`服务器站点

> 请使用校园网连接！

- 主机：172.27.8.24
- 用户名：user010
- 密码：user123456

## 3.将本地数据集上传到 `A100` 服务器

- 本地数据集存放路径：`/home/yuki/.cache/huggingface/lerobot/yuukiii/so100_test03`
- 服务器数据存放路径：`/mnt/data/lerobot/data/inputs`

## 4. 登陆 `A100` 并进入容器

```shell
ssh user010@172.27.8.24
```

```
密码： user123456
```

- **利用脚本进入容器：**

```shell
(base) [user010@localhost ~]$ lerobot

LeroBot 容器管理
--------------------------------
1) 启动并进入容器 (默认)
2) 停止容器
3) 重启容器
4) 查看容器状态和 GPU
5) 删除容器（危险！）
q) 退出
--------------------------------
请选择操作 [1] > 
容器已在运行，正在进入...
root@2064fb6b57c1:~/lerobot# 
```

## 5. 转移数据到容器

```shell
mv /share/inputs/{你的数据}/  ~/.cache/huggingface/lerobot/{你的用户名}/
```

## 6. 激活环境并设置变量

- **激活环境**

```shell
conda activate lerobot-env
```

- **设置用户名和任务名称** *（记得要对应）*

```shell
HF_USER=yuukiii
```

```shell
TASK_NAME=so100_test03
```

## 7. 登陆 `wandb`

```shell
wandb login --relogin
```

## 8. 开始训练

```shell
python lerobot/scripts/train.py \
  --dataset.repo_id=${HF_USER}/${TASK_NAME} \
  --policy.type=act \
  --output_dir=outputs/train/${TASK_NAME} \
  --job_name=${TASK_NAME} \
  --policy.device=cuda \
  --wandb.enable=true
```

**Tips：确认终端开始训练后直接断开校园网即可，他会自动在后台训练！**





# 七. 模型验证

---

## 1. 传输模型文件

1. 打开`FileZilla` 
2. 连接站点（A100）
3. 找到`/mnt/data/lerobot/data/outputs` 文件夹
4. 传输到自己电脑上对应位置
5. 一般只用传输最后一个断点即可

## 2. 新建软链接

新建指向断点模型的`last`软链接

```shell
ln -s 100000 last
```

## 3. 设置变量

```shell
HF_USER=yuukiii
```

## 4. 模型验证

```shell
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --control.type=record \
  --control.fps=30 \
  --control.single_task="lerobot double arm test." \
  --control.repo_id=${HF_USER}/eval_act_so100_test03 \
  --control.tags='["tutorial"]' \
  --control.warmup_time_s=5 \
  --control.episode_time_s=30 \
  --control.reset_time_s=10 \
  --control.num_episodes=15 \
  --control.push_to_hub=false \
  --control.policy.path=outputs/train/act_so100_test03/checkpoints/last/pretrained_model
```

- **control.single_task**：任务描述
- **control.num_episodes**：验证录制集数
- **control.policy.path**：模型路径

以上参数可按需求自行修改。

**快捷操作：**

- `→` ：提前结束该阶段
- `←` ：重新录制该集
- `Esc` ： 退出录制

## 5. [ DecodingError ]

新版本的代码似乎会出现一些编码错误：

<font color="red">***<u>问题：draccus.utils.DecodingError: The fields `device`, `use_amp` are not valid for ACTConfig</u>*** </font>

**解决方案：**

1. 打开 `pretrained_model` 文件夹下的 `config.json` 文件

2. 删除下面两个字段：

    ```json
      "device": "cuda",
      "use_amp": false
    ```

3. 打开 `pretrained_model` 文件夹下的 `train_config.json` 文件

4. 找到 `policy` 部分的下面两个字段，并将其移动到与 `output_dir` 字段同级的位置

    ```json
      "device": "cuda",
      "use_amp": false
    ```

<font color="red">***<u>问题：draccus.utils.DecodingError: `dataset`: The fields `root`, `revision` are not valid for DatasetConfig</u>*** </font>

**解决方案：**

1. 打开 `pretrained_model` 文件夹下的 `train_config.json` 文件
2. 删除 `root` ，`revison` ，`steps`，`run_id`字段
3. 如果后续报其他编码错误，参照此方案继续修改。

## 6. 验证集可视化

- **通过 `Hugging Face` 可视化全部数据集：**

```shell
python lerobot/scripts/visualize_dataset_html.py \
  --repo-id ${HF_USER}/eval_act_so100_test03 \
  --local-files-only 1
```

