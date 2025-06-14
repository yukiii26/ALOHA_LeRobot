# 基于LeRobot框架的通用双臂机器人设计
</br>

# 项目简介
本研究设计并实现一种基于 ACT算法 和 LeRobot 框架 的 低成本 通用双臂机器人，这是一个可远程遥操的系统，用于模仿双手操作，通过对不同任务的少量数据采集并训练，能够较好地执行各种工业上以及生活上的任务，通用性高。</br>
首先先用3D打印的机械臂外壳、舵机及摄像头等搭建好硬件系统，然后修改LeRobot框架实现定制化功能，接着选择“擦盘子”为任务场景，通过遥操进行数据采集，接着把采集好的数据部署到服务器进行训练，最后训练好的模型部署到硬件系统进行验证，验证结果显示擦盘子任务的成功率非常高。


# 项目特点
☆ 低成本复现，双臂+视觉系统成本＜4000RMB，如果加上底盘系统也＜6000RMB（Mobile ALOHA系统成本约3万美金）</br>
☆ 通用性高，各种双臂任务都可以执行，只需要采集少量数据集训练即可（40集左右）</br>
☆ 扩展性强，对 LeRobot框架进行了深度化定制，可支持多机械臂、七自由度机械臂、多摄像头视角，可根据实际需要自由DIY系统


# 项目教程

- 总计1.6w字符的保姆级项目教程可见 [基于 LeRobot 框架 的 通用双臂机器人设计](./基于LeRobot框架的通用双臂机器人设计.md)
- 双臂机器人所需的所有硬件清单可见 [双臂lerobot硬件清单](./双臂lerobot硬件清单.xlsx)，总成本<4000 RMB
- 关于教程中提到的 `libtiff库` 和 `串口调试助手` 也已经上传，有需要可自取



# 效果演示
项目演示视频可参见 [双臂机器人自主擦盘子](./双臂机器人自主擦盘子.mp4)
（可能需要下载到本地才能打开，懒得上传b站或者油管了...）
