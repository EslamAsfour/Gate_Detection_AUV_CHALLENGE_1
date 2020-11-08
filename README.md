# Gate_Detection_AUV_CHALLENGE_1
OpenCV Python code to detect underwater gates

 ## Topics :
***

>  ### 1. Code Steps
>  ### 2. Constraints
>  ### 3. False Detection

***
<br/>

### 1- Code Steps
***
#### - Inputs and Outputs
```python
def DetectGate(Original):
#
#
return {'flag' : __ , 'img' : __ , 'x' : __ , 'y' : __ }

```
<br/>

#### - Get Copy and Filters Stage

```python
    frame = np.copy(Original)
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    cv2.imshow("Original" , frame)
    ret = cv2.adaptiveThreshold(gray,255 ,cv2.ADAPTIVE_THRESH_MEAN_C , cv2.THRESH_BINARY , 9,2)
    median = cv2.medianBlur(ret,5)
    kernel= np.ones((5,5),np.float32)/25
    Filter= cv2.filter2D(median,-1 , kernel)
    edges = cv2.Canny(Filter, 10, 20)
 
```
Frame                        |ret                      |median              
:-------------------------:|:-------------------------:|:-------------------------:
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/img.PNG)  |  ![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/ret.PNG) |![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/ret.PNG) |

Filter                        |Edge                                  
:-------------------------:|:-------------------------:|
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/Filter.PNG)  |  ![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/edges.PNG) |


<br/><br/>
#### - Searching for the Target contour

```python
    contours, hierarchy = cv2.findContours(edges, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
    tempContour =  []
    #Loop throught the Contours to Find the needed ones
    for contour in contours:
              (x, y, w, h) = cv2.boundingRect(contour)
              # We Filter the Contour with the needed Ones
              if h*3 < w and w > 100  and h < 80 and h > 10:
                    tempContour.append(contour)
                    cv2.rectangle(frame, (x, y), (x+w, y+h), (255, 255, 255),2)
 
```
All Contours                       |Target Contours                                  
:-------------------------:|:-------------------------:|
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/Contours.PNG)  |  ![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/Contours2.PNG) |

<br/><br/>

#### - Fitline For the Target Contour
```python 
    # Sort By Arc Lenght and draw the Longest Contour
    cntsSorted = sorted(tempContour, key=lambda x: cv2.arcLength(x,True))
    (x, y, w, h) = cv2.boundingRect(cntsSorted[-1])
    #Draw the Best Contour 
    cv2.rectangle(frame, (x, y), (x+w, y+h), (255, 10, 255),2)
    # then apply fitline() function
    [vx,vy,x,y] = cv2.fitLine(cntsSorted[-1],cv2.DIST_L2,0,0.01,0.01)
    # Now find two extreme points on the line to draw line
    lefty = int((-x*vy/vx) + y)
    righty = int(((gray.shape[1]-x)*vy/vx)+y)
    #Finally draw the line
    cv2.line(frame,(gray.shape[1]-1,righty),(0,lefty),(0,0,0),1)

```

Target Contour Line                      |                               
:-------------------------:|
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/ContLine.PNG)  | 

<br/><br/>

#### - Crop Target Area of the Picture
```python
  # Take the Higher Value 
    if righty < lefty : 
        offset = righty-5
    else :
        offset = lefty-5
    if offset <0   : 
        offset = 0         
    offset = int(offset)
    CheckUnderLine = gray[offset: , : ]

```
Target Contour Line                      |                               
:-------------------------:|
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/Crop.PNG)  | 

<br/><br/>

#### - Canny Filter and HoughLine Function

```python
   edges22 = cv2.Canny(CheckUnderLine,10,20,apertureSize = 3)
   lines = cv2.HoughLinesP(image=edges22,rho=1,theta=np.pi/180, threshold=50, minLineLength=100,maxLineGap=90)
   
```
All HoughLines                    |                               
:-------------------------:|
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/HoughLine.PNG)  | 

<br/><br/>

#### - Why do we crop ?!
``` Let's try the same methods but with the whole image```

Full Img HoughLines                    |                               
:-------------------------:|
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/AllHough.PNG)  | 

<br/><br/>

#### - Detect Two Vertical Lines
```python 
   for i in range(a):
        # Search For the Vertical Lines 
        
        if abs(lines[i][0][1] - lines[i][0][3]) > 5 and abs(lines[i][0][0] - lines[i][0][2]) < 60:
            count = count+1
            TempLine = BotPointThenUpper(lines[i][0])
              # First Line 
            if Vert_Line[0][0] == 0 and  Vert_Line[0][1] == 0:
                #To get The bottom Point
                 Vert_Line[0] = TempLine
                 RefLine = TempLine
                 CountVert1 = CountVert1 + 1
        # Check the variance of the Horizontal axis to get the other vertical line
            elif abs(TempLine[0] - RefLine[0]) > 50:
                Vert_Line[1] =  Vert_Line[1] + TempLine
                CountVert2 = CountVert2 + 1
            else :    
                Vert_Line[0] =  Vert_Line[0] +TempLine
                CountVert1 = CountVert1 + 1


```
Two Verticals       |                               
:-------------------------:|
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/2V.PNG)  | 

<br/><br/>

#### - Average of All vertical lines To get The Best Value
```python
if CountVert1 == 0 or CountVert2 == 0:
        return {'flag' : False , 'img' : Original , 'x' : None, 'y' : None }
    #Create The best two Vertical lines based on the avg
    Vert_Line[0] = (Vert_Line[0]/CountVert1)            
    Vert_Line[1] = (Vert_Line[1]/CountVert2)
    
```

Two Verticals       |                               
:-------------------------:|
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/AvgLine.PNG)  | 

<br/><br/>
#### - Vertical With Horizonal Intersection

```python

    p1_V1 = np.array([ Vert_Line[0][0]  , Vert_Line[0][1]+offset ])
    o1_V1  = np.array([Vert_Line[0][2] ,Vert_Line[0][3]+offset])
    p2_V1 = np.array([gray.shape[1]-1,righty])
    o2_V1 = np.array([0,lefty])
    flag_V1,r_V1 = TwoLineIntersection(o1_V1,p1_V1,p2_V1,o2_V1)
    
    
    def TwoLineIntersection( o1,  p1,  o2,  p2):
    x = o2 - o1
    d1 = p1 - o1
    d2 = p2 - o2

    cross =  d1[0]*d2[1] - d1[1]*d2[0]
    if abs(cross) < 1e-8 : 
        return False ,[0,0]

    t1 = (x[0]* d2[1] - x[1] * d2[0])/cross
    r = o1 + d1 * t1
    return True , r.astype(int)
    
 ```
Gate Detected :sparkling_heart:  |                               
:-------------------------:|
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/Output.PNG)  | 

<br/><br/>

#### - Get and Draw Width

```python

if flag_V1 == True and flag_V2 == True : 
        cv2.line(frame, (r_V1[0],r_V1[1]),(r_V2[0],r_V2[1]), (100, 100, 255), 3, cv2.LINE_AA)
        Width = math.ceil(abs(r_V1[0] - r_V2[0]) *  .2 )   
        if r_V1[0] >  r_V2[0] : 
             cv2.line(frame, (r_V2[0], math.ceil((r_V2[1] +  Vert_Line[1][1]+offset )/2)), (r_V2[0] + Width, math.ceil((r_V2[1] +  Vert_Line[1][1]+offset )/2)), (255, 255,     255), 3, cv2.LINE_AA)
             return {'flag' : True , 'img' : frame , 'x' : r_V2[0] + Width  , 'y' : math.ceil((r_V2[1] +  Vert_Line[1][1]+offset )/2) }

        else : 
             cv2.line(frame, (r_V1[0], math.ceil((r_V1[1] +  Vert_Line[0][1]+offset )/2)), (r_V1[0] + Width, math.ceil((r_V1[1] +  Vert_Line[0][1]+offset )/2)), (255, 255, 255), 3, cv2.LINE_AA)
             return {'flag' : True , 'img' : frame , 'x' : r_V1[0] + Width  , 'y' : math.ceil((r_V1[1] +  Vert_Line[0][1]+offset )/2) }
```
<br/><br/>


### 2. Constraints
#### If we Didn't Find the target Contour :trollface:
```python 
if len(tempContour) == 0 :
        return {'flag' : False , 'img' : Original , 'x' : None, 'y' : None }
```

#### If the Cropped Img is just 25% of the img or lower will be considered *NOT* gat :punch:

```python 
if offset > (gray.shape[0])- (gray.shape[0]*2.5/10) :
        return {'flag' : False , 'img' : Original , 'x' : None, 'y' : None }

```
25%   |                               
:-------------------------:|
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/25.PNG)  | 

<br/><br/>
#### No Lines Detected :punch:
```python 
 lines = cv2.HoughLinesP(image=edges22,rho=1,theta=np.pi/180, threshold=50, minLineLength=100,maxLineGap=90)
    
   # Check if we didnt find any lines in the cropped img
    if lines is None:
        return {'flag' : False , 'img' : Original , 'x' : None, 'y' : None }
```
#### False Far Verticals


```python 

  if math.sqrt( ((r_V2[0]-x)**2)+((r_V2[1]-y)**2) ) <300 : 

```
 Far Verticals  |    Result                           
:-------------------------:|:-------------------------:|
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/far.PNG)  | ![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/Result.PNG)  | 

<br/><br/> 
#### One Vertical Line Detected

```python
 if CountVert1 == 0 or CountVert2 == 0:
        return {'flag' : False , 'img' : Original , 'x' : None, 'y' : None }
```
 One Vertical  |                            
:-------------------------:|
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/OneVer.PNG)  |

<br/><br/> 
### 3. False Detection :trollface::trollface::trollface:

 Far Verticals  |    Result                           
:-------------------------:|:-------------------------:|
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/False.PNG)  | ![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/False1.PNG)  | 

#### :trollface::trollface::trollface::trollface::trollface::trollface:
 ___:trollface:___  |    ___:moyai:___                           
:-------------------------:|:-------------------------:|
![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/What.PNG)  | ![](https://github.com/EslamAsfour/Gate_Detection_AUV_CHALLENGE_1/blob/master/Markdown/Ok.PNG)  | 


<br/><br/> 
<br/><br/> 



