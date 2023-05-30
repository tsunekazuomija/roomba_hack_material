# Pythonコードの読み方

## Purpose
- Pythonで書かれたROSのソースコードの読み方を理解する。  
- Pythonに詳しくない人向け  

## Hands-on

### 1. ROSのソースコードを見つける
- roomba_hackで使うソースコードは https://github.com/matsuolab/roomba_hack に置いてある。  
- その中の catkin_ws/src/ 以下に、ROSのソースコードが置いてある。
- 今回は、catkin_ws/src/navigation_tutorial/scripts/simple_control2.py を読んでみる。  

以下に、simple_control2.pyの中身を載せる。  

```python
#!/usr/bin/env python3
import numpy as np

import rospy
import tf
from geometry_msgs.msg import Twist
from nav_msgs.msg import Odometry

class SimpleController:
    def __init__(self):
        rospy.init_node('simple_controller', anonymous=True)
        
        # Publisher
        self.cmd_vel_pub = rospy.Publisher('/cmd_vel', Twist, queue_size=10)

        # Subscriber
        odom_sub = rospy.Subscriber('/odom', Odometry, self.callback_odom)

        self.x = None
        self.y = None
        self.yaw = None
        while self.x is None:
            rospy.sleep(0.1)

    def callback_odom(self, data):
        self.x = data.pose.pose.position.x
        self.y = data.pose.pose.position.y
        self.yaw = self.get_yaw_from_quaternion(data.pose.pose.orientation)

    def go_straight(self, dis, velocity=0.3):
        vel = Twist()
        x0 = self.x
        y0 = self.y
        while(np.sqrt((self.x-x0)**2+(self.y-y0)**2)<dis):
            vel.linear.x = velocity
            vel.angular.z = 0.0
            self.cmd_vel_pub.publish(vel)
            rospy.sleep(0.1)
        self.stop()

    def turn_right(self, yaw, yawrate=-0.5):
        vel = Twist()
        yaw0 = self.yaw
        while(abs(self.yaw-yaw0)<np.deg2rad(yaw)):
            vel.linear.x = 0.0
            vel.angular.z = yawrate
            self.cmd_vel_pub.publish(vel)
            rospy.sleep(0.1)
        self.stop()

    def turn_left(self, yaw, yawrate=0.5):
        vel = Twist()
        yaw0 = self.yaw
        while(abs(self.yaw-yaw0)<np.deg2rad(yaw)):
            vel.linear.x = 0.0
            vel.angular.z = yawrate
            self.cmd_vel_pub.publish(vel)
            rospy.sleep(0.1)
        self.stop()

    def stop(self):
        vel = Twist()
        vel.linear.x = 0.0
        vel.angular.z = 0.0
        self.cmd_vel_pub.publish(vel)

    def get_yaw_from_quaternion(self, quaternion):
        e = tf.transformations.euler_from_quaternion(
                (quaternion.x, quaternion.y, quaternion.z, quaternion.w))
        return e[2]

if __name__=='__main__':
    simple_controller = SimpleController()
    try:
        simple_controller.go_straight(1.0)
        simple_controller.turn_left(90)
        simple_controller.turn_right(90)
    except rospy.ROSInitException:
        pass
```

### 2. ソースコードの構造を理解する

上のコードの大枠のみ抜き出すと、以下のような構造になっている。

```python
#!/usr/bin/env python3  # 1
import ~  # 2

class SimpleController:  # 3
    def __init__(self):  # 4
        pass
    def callback_odom(self, data):  # 5
        pass
    def go_straight(self, dis, velocity=0.3):
        pass
    def turn_right(self, yaw, yawrate=-0.5):
        pass
    def turn_left(self, yaw, yawrate=0.5):
        pass
    def stop(self):
        pass
    def get_yaw_from_quaternion(self, quaternion):
        pass

if __name__=='__main__':  # 6
    simple_controller = SimpleController()  # 7
    pass
```
それぞれについて、ざっくりと解説する。  
1. shebang(シバン)と呼ばれるもので、このファイルを実行する際に、どのプログラムを使って実行するかを指定する。
    - `#!/usr/bin/env python3` と書いてあるので、このファイルはpython3で実行するのだとコンピュータに教えている。  
    - おまじないとして考えておいて良い。  
    - とりあえず、つくったpythonファイルにはこの一行を書いておく。

1. import文
    - pythonの標準の関数(printなど)だけでは機能が足りないので、別のライブラリからさまざまな関数をインポートしている。
    - このソースコードでは、numpy, rospy, tf, geometry_msgs.msg, nav_msgs.msgというライブラリをインポートしている。
    - rospyというのが、pythonでrosを使うためのライブラリ。

1. classの定義
    - SimpleControllerという名前のオブジェクトを定義している。
    - オブジェクトとは、データとそのデータの振る舞いをまとめたもの。
    - 例えば、以下のようなCarクラスを定義したとする。
    
        ```python
        class Car:
            def __init__(self, color, speed):
                self.color = color
                self.speed = speed
                self.fuel = 100

            def drive(self):
                self.fuel -= 20
                print('drived!')
                print(f'残りの燃料は{self.fuel}リットルです')

            def charge(self):
                self.fuel = 100
                print('charged!')
                print(f'残りの燃料は{self.fuel}リットルです')
            
            def info(self):
                print(f'色は{self.color}です')
                print(f'速度は{self.speed}km/hです')
                print(f'残りの燃料は{self.fuel}リットルです')
        ```

        下のような使い方ができる。

        ```python
        mycar = Car('red', 10000)
        mycar.drive()
        # drived!
        # 残りの燃料は80リットルです

        mycar.drive()
        # drived!
        # 残りの燃料は60リットルです

        mycar.charge()
        # charged!
        # 残りの燃料は100リットルです

        mycar.info()
        # 色はredです
        # 速度は10000km/hです
        # 残りの燃料は100リットルです
        ```

    - pythonのstring型や、int型、list型も、実はオブジェクトである。
        ```python
        # 例
        people = ['Alice', 'Bob', 'Charlie']
        people.append('Dave')

        print(people)
        # ['Alice', 'Bob', 'Charlie', 'Dave']
        ```
        上の例では、list型のオブジェクトpeopleに対して、appendというメソッド(そのオブジェクトが持つ関数)を呼び出し、新しい要素を追加している。

1. コンストラクタの定義  
    - コンストラクタ`__init__()`とは、オブジェクト生成時に呼び出される関数のこと。初期化のための関数というイメージ。

1. メソッドの定義
    - メソッドとは、オブジェクトが持つ関数のこと。
    - classの定義の中では、`self.method_name()`という形で呼び出すことができる。
    - オブジェクトの外から使用するときには、上のCarの例のように、`object_name.method_name()`という形で呼び出すことができる。
    - 定義の第一引数には、必ず`self`を指定する。これは、そのオブジェクト自身を指す。
        - 呼び出すときには`self`は省略する。

1. ファイル実行時に行われる処理
    - このif文の中の処理は、ファイルを直接実行したときにのみ実行される。
    - `__name__`は特殊な変数で、ファイルを直接実行したときには`'__main__'`という値を持つ。
        - importされたときには`__name__`にはファイル名が入るため、このif文の中の処理は実行されない。   
            ```python
            #!/usr/bin/env python3
            print(__name__)
            ```
            とだけ記述したファイルを実行すると、ふるまいが理解しやすいかもしれない。  

### 3. ソースコードのより詳しい理解

上のコードの挙動をより詳しく見ていく
```python
class SimpleController:
    def __init__(self):
        rospy.init_node('simple_controller', anonymous=True)
        '''
        ノードを初期化している。
        このソースコードは'simple_controller'という名前のノードを作成している。
        1つのファイルが一つのノードに対応している。
        '''
        
        # Publisher
        self.cmd_vel_pub = rospy.Publisher('/cmd_vel', Twist, queue_size=10)
        '''
        rospy.Publisher()により、ノードがpublisherとして働くことを宣言している。
        このノードは、/cmd_velという名前のトピックに対して、geometry_msgs/Twist型のメッセージをパブリッシュする。
        queue_sizeは、トピックにメッセージが溜まったときに、溢れないようにするためのもの。

        geometry_msgs/Twist型のメッセージは以下のような構造になっていて、ここではロボットの台車の速度を表している。
        geometry_msgs/Twist
            geometry_msgs/Vector3 linear
                float64 x
                float64 y
                float64 z
            geometry_msgs/Vector3 angular
                float64 x
                float64 y
                float64 z
        '''

        # Subscriber
        odom_sub = rospy.Subscriber('/odom', Odometry, self.callback_odom)
        '''
        rospy.Subscriber()により、ノードがsubscriberとして働くことを宣言している。
        このノードは、/odomという名前のトピックに対して、nav_msgs/Odometry型のメッセージをサブスクライブする。
        このノードがサブスクライブすると、callback_odom()という関数が呼び出される。
        サブスクライブしたメッセージは、callback_odom()の引数として処理される。
        '''
        self.x = None
        self.y = None
        self.yaw = None
        while self.x is None:
            rospy.sleep(0.1)  # rospy.sleep()は、指定した秒数だけ処理を停止する関数。
```

以下に、`__init__`以外の各関数の振る舞いを書く。  

```python
    def callback_odom(self, data):
        '''
        /odomというトピックから、nav_msgs/Odometry型のメッセージをサブスクライブしたときに呼び出される関数。

        nav_msgs/Odometry型のメッセージはrosの標準のメッセージ型なので、
        ggれば出てくる。
        (matsuolab/roomba_hackリポジトリの中で検索して、定義が出てこないものは
        とりあえずggる。)

        nav_msgs/Odometry
            std_msgs/Header header
                uint32 seq
                time stamp
                string frame_id
            string child_frame_id
            geometry_msgs/PoseWithCovariance pose
                geometry_msgs/Pose pose
                    geometry_msgs/Point position
                        float64 x
                        float64 y
                        float64 z
                    geometry_msgs/Quaternion orientation
                        float64 x
                        float64 y
                        float64 z
                        float64 w
                float64[36] covariance
            geometry_msgs/TwistWithCovariance twist
                geometry_msgs/Twist twist
                    geometry_msgs/Vector3 linear
                        float64 x
                        float64 y
                        float64 z
                    geometry_msgs/Vector3 angular
                        float64 x
                        float64 y
                        float64 z
                float64[36] covariance

        余計なものがたくさんついているが、要するに、
        ロボットの位置と速度を格納したmessege型らしい。

        この関数はsubscribeするたびに呼ばれ続けるので、
        常にロボットの現在の位置と速度を取得し,self.x, self.y, self.yawに格納している。
        '''

        self.x = data.pose.pose.position.x　　# ロボットのx座標を取得して格納
        self.y = data.pose.pose.position.y　　# y座標を取得して格納
        self.yaw = self.get_yaw_from_quaternion(data.pose.pose.orientation)
        # ロボットの姿勢を表すクォータニオン(四元数)から、yaw角を取得して格納

    def go_straight(self, dis, velocity=0.3):
        vel = Twist()  # std_msgs/Twist型のメッセージを作成(まだパブリッシュはしない)
        x0 = self.x  # 関数呼び出し時点でのロボットのx座標を取得
        y0 = self.y
        while(np.sqrt((self.x-x0)**2+(self.y-y0)**2)<dis):
            # 現在位置が、関数呼び出し時の位置からdisだけ離れるまで繰り返し
            vel.linear.x = velocity # std_msgs/Twist型のメッセージのlinear.xに速度を代入
            vel.angular.z = 0.0 # 角速度を代入
            self.cmd_vel_pub.publish(vel) 　# ここでパブリッシュ
            rospy.sleep(0.1)
        self.stop()

    def turn_right(self, yaw, yawrate=-0.5): # 略。go_straight()から類推してください
        vel = Twist()
        yaw0 = self.yaw
        while(abs(self.yaw-yaw0)<np.deg2rad(yaw)):
            vel.linear.x = 0.0
            vel.angular.z = yawrate
            self.cmd_vel_pub.publish(vel)
            rospy.sleep(0.1)
        self.stop()

    def turn_left(self, yaw, yawrate=0.5): # 略
        vel = Twist()
        yaw0 = self.yaw
        while(abs(self.yaw-yaw0)<np.deg2rad(yaw)):
            vel.linear.x = 0.0
            vel.angular.z = yawrate
            self.cmd_vel_pub.publish(vel)
            rospy.sleep(0.1)
        self.stop()

    def stop(self): # 略
        vel = Twist()
        vel.linear.x = 0.0
        vel.angular.z = 0.0
        self.cmd_vel_pub.publish(vel)

    def get_yaw_from_quaternion(self, quaternion):
        '''
        クォータニオンからyaw角を計算する関数。
        '''
        e = tf.transformations.euler_from_quaternion(
                (quaternion.x, quaternion.y, quaternion.z, quaternion.w))
        return e[2]
```

以上の処理は、ファイル実行時に読み込まれるのみで、じっさいに動くのは、`if __name__=='main'`以下の部分である。

以下に、ファイル実行時の処理を書く。

```python
if __name__=='__main__': # このファイルを実行したときには、この条件文が成立する。
    simple_controller = SimpleController() # SimpleControllerクラスのオブジェクトを生成
    try:
        simple_controller.go_straight(1.0) # クラスのメソッドを呼び出している。
        simple_controller.turn_left(90)
        simple_controller.turn_right(90)
        # try文は、エラーが起きたときにexcept文の中の処理を実行する。
    except rospy.ROSInitException:
        pass
```

おおきなプログラムを作っていくときには、このように、小さなプログラムを作っていき、それらを組み合わせていくことが多い。

より詳しくいうと、
- task_manager.py : タスクを管理するプログラム。全体の流れが記述されている。service clientやaction clientとして、別のプログラムに細かな処理を依頼する。
- その他のプログラム : task_manager.pyから呼び出されるプログラム。task_manager.pyからの依頼を受けて、実際のタスクを実行する。
    - 上で説明したファイルでは `if __name__=='__main__'`以下に処理の内容が手続的にかかれているが、実際のタスクを実行するときには、serviceのコールバック関数などとして一連の動作をclassの定義の中に組み込むことが多い。

といった構造になっていることが多い。  
(task_manager.pyというのはファイル名の例です)