# BootCamp_Assignment
3주차 영상
<img width="100%" src="https://user-images.githubusercontent.com/105336318/184537121-19a65744-b8cf-4919-b307-eea7a996a48a.gif"/>
3주차 코드
```javascript

```
4주차 영상
<img width="100%" src="https://user-images.githubusercontent.com/105336318/184537270-fb091575-8628-4bee-a0b0-c009c729dee8.gif"/>
4주차 코드
```python3
import sys
import time
import numpy as np
from PyQt5.QtWidgets import QApplication, QMainWindow, QWidget, QVBoxLayout, QLabel, QHBoxLayout, QVBoxLayout
from matplotlib.backends.backend_qt5agg import FigureCanvasQTAgg as FigureCanvas
from matplotlib.backends.backend_qt5agg import NavigationToolbar2QT as NavigationToolbar
from matplotlib.figure import Figure
from pymongo import MongoClient
from PyQt5.QtGui import QIcon, QPixmap, QFont, QFontDatabase
from PyQt5.QtCore import Qt

client = MongoClient("mongodb+srv://<name>:<password>@cluster0.df2ugfu.mongodb.net/?retryWrites=true&w=majority")

db = client['test'] # test라는 이름의 데이터베이스에 접속
  
class MyApp(QMainWindow):
  def __init__(self):
    super().__init__()
    self.main_widget = QWidget()
    self.setCentralWidget(self.main_widget) # 메인 위젯
    self.dynamic_canvas = FigureCanvas(Figure(figsize=(4, 3)))  # 움직이는 그래프를 위한 도화지
    self.dynamic_ax = self.dynamic_canvas.figure.subplots()  # 좌표축 준비

    self.vbox = QVBoxLayout(self.main_widget)  # verticalbox 레이아웃
    self.label1 = QLabel() # 라벨추가
    self.label2 = QLabel()
    self.label3 = QLabel()
    self.label4 = QLabel()
    self.image = QLabel()
    self.Chbox = QHBoxLayout() # 수평 박스

    self.timer = self.dynamic_canvas.new_timer(
        1000, [(self.update_canvas, (), {})])  # 그래프가 변하는 주기를 위한 타이머 준비 (1초, 실행되는 함수)
    self.timer.start()  # 타이머 시작
    
    self.vbox.addWidget(self.label1) # 수직 박스에 위젯 넣기
    self.vbox.addWidget(self.label2)
    self.vbox.addWidget(self.label3)
    self.vbox.addLayout(self.Chbox) # 수직 박스에 수평 박스 넣기
    self.vbox.addWidget(self.dynamic_canvas)

    self.setWindowTitle('실내 미세먼지 농도 측정기')
    self.setGeometry(300, 100, 1000, 600)
    self.show()

  def update_canvas(self):  # 그래프 변화 함수
    self.dynamic_ax.clear()
    count = []
    pm1 = []
    pm2 = []
    pm10 = []
    time = []
    tmp = []

    for i in db['sensors'].find(): # MongoDB에서 데이터 받아오기
      tmp.append(i)
    tmp = tmp[len(tmp)-10:] # 최근 10개의 정보만 받아오기
    for i in range(len(tmp)):
      count.append(i)
      pm1.append(int(tmp[i]["pm1"])) # 마세먼지 농도 받아오기
      pm2.append(int(tmp[i]["pm2"]))
      pm10.append(int(tmp[i]["pm10"]))
      tm = str(tmp[i]["created_at"]) # 시간 받아오기
      time.append(tm[11:]) # 시간, 분, 초만 받기

    self.dynamic_ax.plot(time, pm1, color="black") # 점 찍기(x좌표, y좌표, 뭘로 그릴건지)
    self.dynamic_ax.plot(time, pm2, color='green')
    self.dynamic_ax.plot(time, pm10, color="red")
    self.dynamic_ax.figure.canvas.draw()  # 그리기

    if pm2[-1] <= 15: # 미세먼지 농도에 따른 이미지와 텍스트 변화
      self.image.setPixmap(QPixmap("good.png").scaled(50,50))
      status = '좋음'
    elif pm2[-1] <= 35:
      self.image.setPixmap(QPixmap("soso.png").scaled(50,50))
      status = '보통'
    elif pm2[-1] <= 75:
      self.image.setPixmap(QPixmap("bad.png").scaled(50,50))
      status = '나쁨'
    else:
      self.image.setPixmap(QPixmap("verybad.png").scaled(50,50))
      status = '매우나쁨'

    self.label1.setText(f'현재 PM1 농도는 {pm1[-1]}입니다.') # 라벨에 텍스트 넣기
    self.label2.setText(f'현재 PM2 농도는 {pm2[-1]}입니다.')
    self.label3.setText(f'현재 PM10 농도는 {pm10[-1]}입니다.')
    self.label4.setText(status)

    self.Chbox.addWidget(self.image) # 수평 박스에 위젯 넣기
    self.Chbox.addWidget(self.label4)

    self.label1.setAlignment(Qt.AlignCenter) # 미세먼지 농도 중앙정렬
    self.label2.setAlignment(Qt.AlignCenter)
    self.label3.setAlignment(Qt.AlignCenter)
    self.image.setAlignment(Qt.AlignRight) # 이미지 우측정렬
    self.label4.setAlignment(Qt.AlignLeft) # status 좌측정렬

if __name__ == '__main__':
  app = QApplication(sys.argv)
  fontDB = QFontDatabase() # font를 위한 QFontDatabase선언
  fontDB.addApplicationFont('./NotoSansKR-Regular.otf') # 사용하고자 하는 font등록
  app.setFont(QFont('NotoSansKR-Regular')) # app에다 적용
  ex = MyApp()
  sys.exit(app.exec_())
```
