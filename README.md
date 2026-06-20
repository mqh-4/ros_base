#ros2 核心概念
节点 Node：ROS2 最小执行单元，一个功能对应一个节点，每个节点独立进程，通过 DDS 通信，解耦。
话题 Topic（异步单向通信）：发布者 (Publisher) 发消息，订阅者 (Subscriber) 接收，一对多、多对多，适合持续数据流（激光雷达、相机图像）。
服务 Service（同步双向通信）：客户端发请求、服务端返回应答，一问一答阻塞，适合单次指令（开关设备、获取机器人坐标）。
动作 Action（异步双向通信）：目标 + 反馈 + 结果，长耗时任务（导航、机械臂运动、路径跟踪），可中途取消。
rclcpp：ROS2 C++ 标准客户端库，封装 DDS 底层通信，提供节点、话题、服务、动作 API。
工作空间 Workspace：存放源码 src、编译目录 build/install/log，标准目录结构。
colcon：ROS2 统一编译工具，替代 catkin，支持多语言包并行编译。
DDS = Data Distribution Service，数据分发服务，是一套工业级实时分布式通信标准（OMG 组织制定），专门用来解决多进程 / 多设备之间实时收发数据。
ROS 2 完全基于 DDS 做底层通信；ROS 1 没有 DDS，用的是自定义 TCP/UDP 通信（roscore），这是 ROS1 和 ROS2 最核心区别。
核心定位：机器人多节点、多机、多传感器实时数据传输中间件，负责：话题发布订阅、服务、动作、参数、节点发现。
DDS 核心设计思想：发布 - 订阅（Publish-Subscribe）
发布者 Publisher：发送数据（相机图像、激光雷达、速度指令）
订阅者 Subscriber：接收对应话题数据
数据域 Domain：同一数字域内的节点才能互相发现、通信（默认 domain_id=0）

#话题通信   发布者  +   订阅者
  发布者：
  ```
      #include "rclcpp/rclcpp.hpp"
      #include "std_mags/msg/string.hpp"
      #include <chrono>

      class Talker: public rclcpp:: Node{
        public :
            Talker(): Node("talker_node"){
               pub_=this->create_publisher<std_msgs::msg::String>("chatter",10);
               timer_=this->create_wall_timer(chrono_literals::500ms,std::bind(&Talker::timer_cb,this));
              cnt_=0;
}
       private:
              void timer_cb(){
                  auto msg = std_msgs::msg::String();
                  msg.data = "hello ros2 cnt:" + std::to_string(ccnt++);
                  RCLCPP_INFO(this->get_logger(),"fabu :%s",msg.data.c_str());
                  pub_->publish(msg);
}
            rclcpp::Publisher<std_msgs::msg::String>::SharedPtr pub_;
            rclcpp::TimeBase::SharedPtr timer_;
            int cnt_;
 };
          int main(int argc,int** argv){
              rclcpp::init(argc,argv);
              rclcpp::spin(std::make_shared<Talker>());
              rclcpp::shutdown();
              return 0;
}              
```
              
