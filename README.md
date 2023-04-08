# Ground-Truth
## 🌞 프로젝트 기획 이유

당시 학원 다닐 때의 맡게된 기업프로젝트로, 오토 라벨링이라는 서비스를 맡게 되었다. 

## 🤩 프로젝트 개발 목표 및 내용

- ***회원가입과 로그인***
    - 회원가입을 하고 로그인해야 사용자가 라벨링하려고 하는 물체의 업로드등의 모든 기능을 사용할 수 있게 하였다.
- ***라벨링***
    - 동영상을 업로드해야 한다.
    - 동영상을 업로드하고 이를 bbox라고 하여 물체가 어디있는지 사용자가 표시해줘야 한다.
    - 동영상이 클 수록 표시해줘야 하는 박스 갯수가 늘어난다. 이는 최대 10개로만 제한하였다.
- ***결과물***
    - 라벨링한 결과물이 xml로 나온다. 이를 여러 동영상에서도 표시할 수 있게 하였다.

## **1. 사용 기술**

- **언어** : Python, Flask, HTML, CSS, Javascript, SQL
- **DB** : Oracle
- **Framworks** : Bootstrap
- **IDEs** : Vscode

## 2**. 프로젝트 일정**

![https://user-images.githubusercontent.com/89984853/198654840-1af0df68-a262-4d93-82a4-ae77b238908f.png](https://user-images.githubusercontent.com/89984853/198654840-1af0df68-a262-4d93-82a4-ae77b238908f.png)

## 3**. 구현 화면**

[프로젝트2차 영상.mp4](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1336fdc4-9a6e-4e01-ab67-275c79fc80ff/프로젝트2차_영상.mp4)

### 페이지 소개

→ 프로젝트의 페이지는 **대표적으로 세 개로 이루어져** 있다. 구현 화면은 ppt사진으로 대신한다. 

### 1) ***첫번째 페이지***

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/82072061-7a6d-4fce-9db5-82e538f07a81/Untitled.png)

그리고 처음 들어간 사용자들이 어떻게 사용하는지 한 눈에 알아볼 수 있게끔 사용 설명서를 넣었다. 이에 대한 영상이 나오게 동영상 파일도 넣었다. 그리고 ***라벨링하고자 하는 영상을 업로드***하면 이의 이미지를 프레임마다 저장하고 처리가 어느정도 되었는지 프로그래스 바로 표시하게 하였다. 사용자로 하여금 기다리는 것이 지루하지 않도록 시각적으로 확인되게 하였다.

### 2) ***두번째 페이지***

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7ae541e6-0e30-4fa5-9d3f-7e3f57fa34e1/Untitled.png)

영상의 대표적인 이미지를 추출하여 사용자에게 나타나게 하였다. 아래쪽에는 총 프레임 수와 대표 프레임수를 넣었다. 그리고 다음 버튼을 누르면 ***대표프레임을 불러와서 사용자 본인이 바운드 박스를 치게*** 한다. 이 말은 동영상에서 어떠한 객체를 추적하려고 하는지 사용자로 하여 표시하게 하였다. 그리고 그 좌표값을 저장하여 다음페이지로 넘겨준다.

### 3) ***세번째 페이지***

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f8c5dac4-1eed-4e96-b24f-3a5760980760/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/760d5572-f681-417c-b174-d63d76969009/Untitled.png)

***전체 이미지와 좌표값들에 대한 압축 파일을 다운***로드 받을 수 있다. 

### 딥러닝관련 소개

- **전반적인 딥러닝 코드**
    
    ```python
    # 1단계. 프레임만 추출/저장
    import cv2, sys, os
    import imutils
    
    video_src = 0 # 비디오 파일과 카메라 선택 ---②
    video_src = "./sample/22.mp4"
    cap = cv2.VideoCapture(video_src)
    
    fps = cap.get(cv2.CAP_PROP_FPS) # 프레임 수 구하기
    delay = int(1000/fps)
    
    imgs = []
    # 캡처
    success = False
    count = 1
    
    while cap.isOpened():
        ret, frame = cap.read()
        if not ret :
            print("이미지 읽기 실패 또는 다 읽음")
    
            # 동영상 캡처 객체를 종료
            cap.release()
    
            # 열린 창을 닫아준다
            cv2.destroyAllWindows()
            break
        frame = imutils.resize(frame, width=500, height = 500)
        if not ret:
            print('Cannot read video file')
            break
            
        img_draw = frame.copy()
         
        if success:
            cv2.imwrite("./vid/image{}.jpg".format(count), img_draw, params=[cv2.IMWRITE_PNG_COMPRESSION, 0])
            imgs.append(('./vid/image{}.jpg'.format(count)))
            print("./vid/image{}.jpg 파일을 저장".format(count))
            # 크기는 frame크기와 같게
            count +=1
        # 추적 실패
        if ret:
            success=True
        else:
            success=False
    ```
    
    ```python
    2단계. 유사도 비교
    # 유사도 비교(히스토그램)
    # 유사도 수 30 이상이면 유사도 수치 낮춤. 10 미만이면 높임
    hists = []
    # for j in range(len(imgs)):
    cut = 0
    cutimgnum = []
    for i in range(cut, len(imgs)) :
        #---① 각 이미지를 HSV로 변환
        hsv = cv2.cvtColor(cv2.imread(imgs[i]), cv2.COLOR_BGR2HSV)
        #---② H,S 채널에 대한 히스토그램 계산
        hist = cv2.calcHist([hsv], [0,1], None, [180,256], [0,180,0, 256])
        #---③ 0~1로 정규화
        cv2.normalize(hist, hist, 0, 1, cv2.NORM_MINMAX)
        hists.append(hist)
    
        query = hists[cut]
        correl = cv2.HISTCMP_CORREL
        ret = cv2.compareHist(query, hist, correl)
    
        
        if ret > 0.80:
            print("유사도 ===> ", "img%d:%7.2f"% (i+1 , ret))
            image = "img%d:%7.2f"% (i+1 , ret)
        else:
            cut = i
            i = cut
            cutimgnum.append(i+1)
            print("유사도 ===> ", "img%d:%7.2f"% (i+1 , ret))
            print("유사도 딸림")
            
    if len(cutimgnum) < 10:
        print('******유사도 다시 비교******')
        hists = []
        # for j in range(len(imgs)):
        cut = 0
        cutimgnum = []
        for i in range(cut, len(imgs)) :
            #---① 각 이미지를 HSV로 변환
            hsv = cv2.cvtColor(cv2.imread(imgs[i]), cv2.COLOR_BGR2HSV)
            #---② H,S 채널에 대한 히스토그램 계산
            hist = cv2.calcHist([hsv], [0,1], None, [180,256], [0,180,0, 256])
            #---③ 0~1로 정규화
            cv2.normalize(hist, hist, 0, 1, cv2.NORM_MINMAX)
            hists.append(hist)
    
            query = hists[cut]
            correl = cv2.HISTCMP_CORREL
            ret = cv2.compareHist(query, hist, correl)
                    #                 ret = ret/query        #비교대상으로 나누어 1로 정규화
    
            if ret > 0.90:
                print("유사도 ===> ", "img%d:%7.2f"% (i+1 , ret))
                image = "img%d:%7.2f"% (i+1 , ret)
            else:
                cut = i
                i = cut
                cutimgnum.append(i+1)
                print("유사도 ===> ", "img%d:%7.2f"% (i+1 , ret))
                print("유사도 딸림")
                
    if len(cutimgnum) > 30:
        print('******유사도 다시 비교******')
        hists = []
        # for j in range(len(imgs)):
        cut = 0
        cutimgnum = []
        for i in range(cut, len(imgs)) :
            #---① 각 이미지를 HSV로 변환
            hsv = cv2.cvtColor(cv2.imread(imgs[i]), cv2.COLOR_BGR2HSV)
            #---② H,S 채널에 대한 히스토그램 계산
            hist = cv2.calcHist([hsv], [0,1], None, [180,256], [0,180,0, 256])
            #---③ 0~1로 정규화
            cv2.normalize(hist, hist, 0, 1, cv2.NORM_MINMAX)
            hists.append(hist)
    
            query = hists[cut]
            correl = cv2.HISTCMP_CORREL
            ret = cv2.compareHist(query, hist, correl)
                    #                 ret = ret/query        #비교대상으로 나누어 1로 정규화
    
            if ret > 0.60:
                print("유사도 ===> ", "img%d:%7.2f"% (i+1 , ret))
                image = "img%d:%7.2f"% (i+1 , ret)
            else:
                cut = i
                i = cut
                cutimgnum.append(i+1)
                print("유사도 ===> ", "img%d:%7.2f"% (i+1 , ret))
                print("유사도 딸림")
                
    if len(cutimgnum) > 30:
        print('******유사도 다시 비교******')
        hists = []
        # for j in range(len(imgs)):
        cut = 0
        cutimgnum = []
        for i in range(cut, len(imgs)) :
            #---① 각 이미지를 HSV로 변환
            hsv = cv2.cvtColor(cv2.imread(imgs[i]), cv2.COLOR_BGR2HSV)
            #---② H,S 채널에 대한 히스토그램 계산
            hist = cv2.calcHist([hsv], [0,1], None, [180,256], [0,180,0, 256])
            #---③ 0~1로 정규화
            cv2.normalize(hist, hist, 0, 1, cv2.NORM_MINMAX)
            hists.append(hist)
    
            query = hists[cut]
            correl = cv2.HISTCMP_CORREL
            ret = cv2.compareHist(query, hist, correl)
                    #                 ret = ret/query        #비교대상으로 나누어 1로 정규화
    
            if ret > 0.50:
                print("유사도 ===> ", "img%d:%7.2f"% (i+1 , ret))
                image = "img%d:%7.2f"% (i+1 , ret)
            else:
                cut = i
                i = cut
                cutimgnum.append(i+1)
                print("유사도 ===> ", "img%d:%7.2f"% (i+1 , ret))
                print("유사도 딸림")
    if len(cutimgnum) > 30:
        print('******유사도 다시 비교******')
        hists = []
        # for j in range(len(imgs)):
        cut = 0
        cutimgnum = []
        for i in range(cut, len(imgs)) :
            #---① 각 이미지를 HSV로 변환
            hsv = cv2.cvtColor(cv2.imread(imgs[i]), cv2.COLOR_BGR2HSV)
            #---② H,S 채널에 대한 히스토그램 계산
            hist = cv2.calcHist([hsv], [0,1], None, [180,256], [0,180,0, 256])
            #---③ 0~1로 정규화
            cv2.normalize(hist, hist, 0, 1, cv2.NORM_MINMAX)
            hists.append(hist)
    
            query = hists[cut]
            correl = cv2.HISTCMP_CORREL
            ret = cv2.compareHist(query, hist, correl)
                    #                 ret = ret/query        #비교대상으로 나누어 1로 정규화
    
            if ret > 0.40:
                print("유사도 ===> ", "img%d:%7.2f"% (i+1 , ret))
                image = "img%d:%7.2f"% (i+1 , ret)
            else:
                cut = i
                i = cut
                cutimgnum.append(i+1)
                print("유사도 ===> ", "img%d:%7.2f"% (i+1 , ret))
                print("유사도 딸림")
    ```
    
    ```python
    3단계. 대표이미지에 roi 체크
    import cv2
    import numpy as np
    bbox1 = []
    point = ""
    isDragging = False                      # 마우스 드래그 상태 저장 
    x0, y0, w, h = -1,-1,-1,-1              # 영역 선택 좌표 저장
    blue, red = (255,0,0),(0,0,255)         # 색상 값 
    bbox2 = []
    win_name = 'Tracking APIs'
    def indent(elem, level=0):
        i = "\n" + level*"  "
        if len(elem):
            if not elem.text or not elem.text.strip():
                elem.text = i + "  "
                
            if not elem.tail or not elem.tail.strip():
                elem.tail = i
                
            for elem in elem:
                indent(elem, level+1)
                
            if not elem.tail or not elem.tail.strip():
                elem.tail = i
                
        else:
            if level and (not elem.tail or not elem.tail.strip()):
                elem.tail = i
    
    def onMouse(event,x,y,flags,param):     # 마우스 이벤트 핸들 함수  ---①
        global isDragging, x0, y0, img      # 전역변수 참조
        if event == cv2.EVENT_LBUTTONDOWN:  # 왼쪽 마우스 버튼 다운, 드래그 시작 ---②
            isDragging = True
            x0 = x
            y0 = y
        elif event == cv2.EVENT_MOUSEMOVE:  # 마우스 움직임 ---③
            if isDragging:                  # 드래그 진행 중
                img_draw = img.copy()       # 사각형 그림 표현을 위한 이미지 복제
                cv2.rectangle(img_draw, (x0, y0), (x, y), blue, 2) # 드래그 진행 영역 표시
                cv2.imshow('img', img_draw) # 사각형 표시된 그림 화면 출력
        elif event == cv2.EVENT_LBUTTONUP:  # 왼쪽 마우스 버튼 업 ---④
            if isDragging:                  # 드래그 중지
                isDragging = False          
                w = x - x0                  # 드래그 영역 폭 계산
                h = y - y0                  # 드래그 영역 높이 계산
                print("x:%d, y:%d, w:%d, h:%d" % (x0, y0, w, h))
                if w > 0 and h > 0:         # 폭과 높이가 양수이면 드래그 방향이 옳음 ---⑤
                    img_draw = img.copy()   # 선택 영역에 사각형 그림을 표시할 이미지 복제
                    # 선택 영역에 빨간 사각형 표시
                    cv2.rectangle(img_draw, (x0, y0), (x, y), red, 2) 
                    cv2.imshow('ROI', img_draw) # 빨간 사각형 그려진 이미지 화면 출력
                    roi = img[y0:y0+h, x0:x0+w] # 원본 이미지에서 선택 영영만 ROI로 지정 ---⑥
                    cv2.imshow('cropped', roi)  # ROI 지정 영역을 새창으로 표시
                    cv2.moveWindow('cropped', 0, 0) # 새창을 화면 좌측 상단에 이동
                    cv2.imwrite('./cropped.jpg', roi)   # ROI 영역만 파일로 저장 ---⑦
                    print("croped.")
                    filename = ('image{}'.format(i))
                            # 크기는 frame크기와 같게
                    print("파일이름 > ", filename)
                    width = 500
                    height = 500
    
    for i in cutimgnum:
        
        img = cv2.imread('./vid/image{}.jpg'.format(i), cv2.IMREAD_COLOR)
        roi = cv2.selectROI(win_name, img, False)
        bbox2.append(roi)
        cv2.imshow('img', img)
        cv2.setMouseCallback('img', onMouse) # 마우스 이벤트 등록 ---⑧
        cv2.waitKey()
        cv2.destroyAllWindows()
    ```
    
    ```python
    4단계. 전체 프레임에 ROI, 이미지와 XML 파일 저장
    import cv2, sys, os
    import imutils
    import numpy as np
    import matplotlib.pylab as plt
    # cv2 버전 3.4.15로 낮춰서 할 것
    # XML파일 생성 from import
    from xml.etree.ElementTree import Element, SubElement, ElementTree, dump
    
    # goturn.caffemodel / goturn.prototxt 다운받아서 실행하는 ipynb와 같은 폴더에 넣기
    # https://github.com/Mogball/goturn-files
    
    if not (os.path.isfile('goturn.caffemodel') and os.path.isfile('goturn.prototxt')):
        errorMsg = '''
        Could not find GOTURN model in current directory.
        Please ensure goturn.caffemodel and goturn.prototxt are in the current directory
        '''
    
        print(errorMsg)
        sys.exit()
    # 트랙커 객체 생성자 함수 리스트 ---①
    trackers = [cv2.TrackerBoosting_create, # 0
                cv2.TrackerMIL_create, # 1
                cv2.TrackerKCF_create, # 2
                cv2.TrackerTLD_create, # 3 
                cv2.TrackerMedianFlow_create, # 4
                cv2.TrackerGOTURN_create, # 5
                cv2.TrackerMOSSE_create, # 6
                cv2.TrackerCSRT_create] # 7
    trackerIdx = 7
        
    # 트랙커 생성자 함수 선택 인덱스
    tracker = None
    isFirst = True
    
    video_src = 0 # 비디오 파일과 카메라 선택 ---②
    video_src = "./sample/92.mp4"
    cap = cv2.VideoCapture(video_src)
    
    fps = cap.get(cv2.CAP_PROP_FPS) # 프레임 수 구하기
    delay = int(1000/fps)
    win_name = 'Tracking APIs'
    imgs = [] 
    
    # 캡처
    success = False
    count = 1
    co = 0
    # XML 줄 예쁘게
    def indent(elem, level=0):
        i = "\n" + level*"  "
        if len(elem):
            if not elem.text or not elem.text.strip():
                elem.text = i + "  "
                
            if not elem.tail or not elem.tail.strip():
                elem.tail = i
                
            for elem in elem:
                indent(elem, level+1)
                
            if not elem.tail or not elem.tail.strip():
                elem.tail = i
                
        else:
            if level and (not elem.tail or not elem.tail.strip()):
                elem.tail = i
    
        
    while cap.isOpened():
         
        ret, frame = cap.read()
        if not ret :
            print("이미지 읽기 실패 또는 다 읽음")
    
            # 동영상 캡처 객체를 종료
            cap.release()
    
            # 열린 창을 닫아준다
            cv2.destroyAllWindows()
            break
        frame = imutils.resize(frame, width=500)
        if not ret:
            print('Cannot read video file')
            break
            
        img_draw = frame.copy()
        
        # 트랙커 생성 안된 경우 "Press the Space to set ROI!!"글자 띄움
        if tracker is None: 
            cv2.putText(img_draw, "Press the Space to set ROI!!", \
                (100,80), cv2.FONT_HERSHEY_SIMPLEX, 0.75,(0,0,255),2,cv2.LINE_AA)
        
        # 트랙커 생성된 경우
        else:
            ok, bbox = tracker.update(frame)   # 새로운 프레임에서 추적 위치 찾기 ---③
            (x,y,w,h) = bbox
            
            # 추적 성공
            if ok: 
                cv2.rectangle(img_draw, (int(x), int(y)), (int(x + w), int(y + h)), \
                              (0,255,0), 2, 1)
                print("포인터 위치{}:".format(count),int(x), int(y), int(x + w), int(y + h))
                #print("bbox의 x위치:",bbox[0])
                
                # 추적 성공하면 캡처해서 저장
                if success:
                    cv2.imwrite("./vi/image{}.jpg".format(count), img_draw, params=[cv2.IMWRITE_PNG_COMPRESSION, 0])
                    imgs.append(cv2.imread('./vi/image{}.jpg'.format(count)))
                    print("./vi/image{}.jpg 파일을 저장".format(count))
                    if cv2.waitKey(10) == 27:
                        break
          
                    
                    # bbox 좌표값을 가진 XML파일 생성 
                    filename = 'image{}'.format(count)
    
                    # 크기는 frame크기와 같게
                    width = 500
                    height = 500
    
                    # bbox 좌표
                    point1 = (int(x), int(y))
                    point2 = (int(x+w), int(y+h))
                    
                    # 객체이름
                    label = 'crown'
    
                    root = Element('annotation')
                    SubElement(root, 'folder').text = 'annotation'
                    SubElement(root, 'filename').text = filename + '.jpg'
                    SubElement(root, 'path').text = './annotation/' +  filename + '.jpg'
                    source = SubElement(root, 'source')
                    SubElement(source, 'database').text = 'Unknown'
    
                    size = SubElement(root, 'size')
                    SubElement(size, 'width').text = str(width)
                    SubElement(size, 'height').text = str(height)
                    SubElement(size, 'depth').text = '1'
    
                    SubElement(root, 'segmented').text = '0'
    
                    obj = SubElement(root, 'object')
                    SubElement(obj, 'name').text = label
                    SubElement(obj, 'pose').text = 'Unspecified'
                    SubElement(obj, 'truncated').text = '0'
                    SubElement(obj, 'difficult').text = '0'
                    bbox = SubElement(obj, 'bndbox')
                    SubElement(bbox, 'xmin').text = str(point1[0])
                    SubElement(bbox, 'ymin').text = str(point1[1])
                    SubElement(bbox, 'xmax').text = str(point2[0])
                    SubElement(bbox, 'ymax').text = str(point2[1])
                    
    
                    indent(root)
                    
                    tree = ElementTree(root)
                    tree.write('./annotation/' + filename +'.xml')
                    print("./annotation/image{}.xml 파일을 저장".format(count))
                                  
                    count +=1
    #             if count == cutimgnum:
    #                     roi1=(bbox1[0][0],bbox1[0][1],bbox1[0][2],bbox1[0][3])
    
                    print(cap.get(cv2.CAP_PROP_POS_FRAMES))
                    if cutimgnum[co]==int(cap.get(cv2.CAP_PROP_POS_FRAMES)):
                        roi=bbox2[co]
                        co+=1                    
                        if roi[2] and roi[3]:          
                            tracker = trackers[trackerIdx]()    #트랙커 객체(박스) 생성 ---⑤
                            isInit = tracker.init(frame, roi)
        
            # 추적 실패
            else : 
                cv2.putText(img_draw, "Tracking fail.", (100,80), \
                            cv2.FONT_HERSHEY_SIMPLEX, 0.75,(0,0,255),2,cv2.LINE_AA)
                
            
        trackerName = tracker.__class__.__name__
        cv2.putText(img_draw, str(trackerIdx) + ":"+trackerName , (100,20), \
                     cv2.FONT_HERSHEY_SIMPLEX, 0.75, (0,255,0),2,cv2.LINE_AA)
        
        
        
        cv2.imshow(win_name, img_draw)
        key = cv2.waitKey(delay) & 0xff
        
        if ret:
            success=True
        else:
            success=False
        
        if(video_src != 0 and isFirst): 
            
            # 처음부터 시작 하지마
            isFirst = False 
            
            roi = cv2.selectROI(win_name, frame, False)  # 초기 객체 위치 설정 
            print("초기 roi값:",roi)
    
            #위치 설정 값 있는 경우
            if roi[2] and roi[3]:         
                tracker = trackers[trackerIdx]()    #트랙커 객체(박스) 생성 ---⑤
                print("트랙커:",tracker)
                isInit = tracker.init(frame, roi)
                    
            if cutimgnum[co]==count:
                roi=bbox2[co]
                co+=1                    
                if roi[2] and roi[3]:          
                    tracker = trackers[trackerIdx]()    #트랙커 객체(박스) 생성 ---⑤
                    print("@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@")
                    isInit = tracker.init(frame, roi)
                            
        elif key in range(48, 56): # 0~7 숫자 입력   ---⑥
            trackerIdx = key-48     # 선택한 숫자로 트랙커 인덱스 수정
            
            if bbox is not None:
                tracker = trackers[trackerIdx]() # 선택한 숫자의 트랙커 객체 생성 ---⑦
                isInit = tracker.init(frame, bbox) # 이전 추적 위치로 추적 위치 초기화
                
        elif key == 27 : 
            break
    else:
        print( "Could not open video")
    cap.release()
    cv2.destroyAllWindows()
    ```
    

### 1) 초기의 tracker api 사용

초기에는 물체를 탐지하는 tracker api를 써서 물체탐지하는 기능을 정확도 높은 모델로 물체를 추적하게 하였다. 그러나 여러 동영상의 물체를 추적하게 했을 경우, 물체를 잘 추적하지 못했다. 그 이유는 배경과 물체의 색이 비슷할 경우 같은 물체라고 인식하였기 때문이다. 

### 2) 모델 변경

이 후에 정확도가 높지 않은 것을 확인하여 여러 모델을 썼다. 나중에 yolo모델을 써서 정확도를 높였다. 그리고 회사의 피드백을 받아 유사도를 비교하였다. 그리고 히스토그램을 사용하여 동영상에서 bbox를 쳤을 때 물체를 잘 따라갈 수 있게 하였다. 

그리고 정확도가 어느 정도 되는지 알고 싶어서 ***프레임을 뽑아서 저장***한 후에 이를 학습시켜 정확도와 정밀도를 뽑아서 어느 정도의 정확도가 높다는 것을 알게 되었다. 

그리고 동영상에 bbox가 물체를 잘 따라갈 수 있도록 ***대표이미지를 여러 개 뽑아내어 관심 영역을 지정***하였다. 그리고 학습한 결과물을 사용자들이 사용할 수 있도록 ***지정한 좌표값을 xml파일로 저장***하였다. 이는 사용자들이 다운받아서 XML파일을 이용해 학습시킬 수 있다.
