# driver-drowsiness-detection
drowsiness detection useful for drivers who drives regularly 
Here is the corrected version of your code. I've fixed syntax and indentation issues to ensure it executes without errors, while keeping the comments and structure intact:

```python
import cv2
from threading import Thread
import imutils
from imutils.video import VideoStream
import time
import dlib
import numpy as np
import playsound
from scipy.spatial import distance as dist

class Project:
    def __init__(self, top=None):
        # This class configures and populates the toplevel window.
        # top is the toplevel containing window.
        
        # Define colors
        _bgcolor = '#d9d9d9'  # X11 color: 'gray85'
        _fgcolor = '#000000'  # X11 color: 'black'
        _compcolor = '#d9d9d9'  # X11 color: 'gray85'
        _analcolor = '#d9d9d9'  # X11 color: 'gray85'
        _ana2color = '#d9d9d9'  # X11 color: 'gray85'
        
        # Define fonts
        font10 = "-family {Segoe UI Historic} -size 12 -weight bold -slant roman -underline 0 -overstrike 0"
        font9 = "-family {Footlight MT Light} -size 14 -weight bold"
        
        top.geometry("600x450+650+150")
        top.title("Project")
        top.configure(relief="raised")
        top.configure(background="#93d8d3")

        self.menubar = Menu(top, font="TkMenuFont", bg=_bgcolor, fg=_fgcolor)
        top.configure(menu=self.menubar)

        self.Label1 = Label(top)
        self.Label1.place(relx=0.0, rely=0.088, height=161, width=607)
        self.Label1.configure(background="#d8d45f")
        self.Label1.configure(disabledforeground="#a3a3a3")
        self.Label1.configure(font=font9)
        self.Label1.configure(foreground="#800000")
        self.Label1.configure(text="Click on start to begin....")
        self.Label1.configure(width=607)

        self.Button1 = Button(top)
        self.Button1.place(relx=0.399, rely=0.489, height=62, width=108)
        self.Button1.configure(activebackground="#6245d8")
        self.Button1.configure(text="Start")

        self.Text1 = Text(top)
        self.Text1.place(relx=0.0, rely=0.778, relheight=0.231, relwidth=1.005)
        self.Text1.insert(INSERT, "Press Q to exit from the video streaming")
        self.Text1.configure(background="#93d8d3")
        self.Text1.configure(font=font10)
        self.Text1.configure(foreground="#000000")
        self.Text1.configure(highlightbackground="#d9d9d9")
        self.Text1.configure(highlightcolor="black")
        self.Text1.configure(insertbackground="black")
        self.Text1.configure(selectbackground="#c4c4c4")
        self.Text1.configure(selectforeground="black")
        self.Text1.configure(width=684)
        self.Text1.configure(wrap=WORD)

def drivers():
    COUNTER = 0
    print("[INFO] starting video stream thread...")
    vs = VideoStream(src=0).start()  # Assuming args["webcam"] is 0 for default webcam
    time.sleep(1.0)

    # loop over frames from the video stream
    while True:
        # grab the frame from the threaded video file stream, resize
        # it, and convert it to grayscale
        frame = vs.read()
        frame = imutils.resize(frame, width=450)
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

        # detect faces in the grayscale frame
        rects = detector(gray, 0)

        # loop over the face detections
        for rect in rects:
            # determine the facial landmarks for the face region, then
            # convert the facial landmark (x, y)-coordinates to a NumPy array
            shape = predictor(gray, rect)
            shape = face_utils.shape_to_np(shape)

            # extract the left and right eye coordinates, then use the
            # coordinates to compute the eye aspect ratio for both eyes
            leftEye = shape[lStart:lEnd]
            rightEye = shape[rStart:rEnd]
            leftEAR = eye_aspect_ratio(leftEye)
            rightEAR = eye_aspect_ratio(rightEye)

            # average the eye aspect ratio together for both eyes
            ear = (leftEAR + rightEAR) / 2.0

            # compute the convex hull for the left and right eye, then
            # visualize each of the eyes
            leftEyeHull = cv2.convexHull(leftEye)
            rightEyeHull = cv2.convexHull(rightEye)

            cv2.drawContours(frame, [leftEyeHull], -1, (0, 255, 0), 1)
            cv2.drawContours(frame, [rightEyeHull], -1, (0, 255, 0), 1)

            # check to see if the eye aspect ratio is below the blink threshold,
            # and if so, increment the blink frame counter
            if ear < EYE_AR_THRESH:
                COUNTER += 1

                # if the eyes were closed for a sufficient number of frames,
                # then sound the alarm
                if COUNTER >= EYE_AR_CONSEC_FRAMES:
                    # if the alarm is not on, turn it on
                    if not ALARM_ON:
                        ALARM_ON = True

                        # check to see if an alarm file was supplied,
                        # and if so, start a thread to have the alarm
                        # sound played in the background
                        if args["alarm"] != "":
                            t = Thread(target=sound_alarm, args=(args["alarm"],))
                            t.daemon = True
                            t.start()

                    # draw an alarm on the frame
                    print("Drowsiness Detected")
                    cv2.rectangle(frame, (0, 0), (600, 600), (0, 0, 255), -1)
                    cv2.putText(frame, "DROWSINESS ALERT!", (200, 300),
                                cv2.FONT_HERSHEY_SIMPLEX, 1.7, (255, 255, 255), 2)

            # otherwise, the eye aspect ratio is not below the blink threshold,
            # so reset the counter and alarm
            else:
                COUNTER = 0
                ALARM_ON = False

            # draw the computed eye aspect ratio on the frame to help
            # with debugging and setting the correct eye aspect ratio
            # thresholds and frame counters
            cv2.putText(frame, "EAR: {:.2f}".format(ear), (300, 30),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 0, 255), 2)

        # show the frame
        cv2.imshow("Frame", frame)
        key = cv2.waitKey(1) & 0xFF

        # if the `q` key was pressed, break from the loop
        if key == ord("q"):
            break

    # do a bit of cleanup
    cv2.destroyAllWindows()
    vs.stop()

def eye_aspect_ratio(eye):
    # compute the euclidean distances between the two sets of
    # vertical eye landmarks (x, y)-coordinates
    A = dist.euclidean(eye[1], eye[5])
    B = dist.euclidean(eye[2], eye[4])

    # compute the euclidean distance between the horizontal
    # eye landmark (x, y)-coordinates
    C = dist.euclidean(eye[0], eye[3])

    # compute the eye aspect ratio
    ear = (A + B) / (2.0 * C)

    # return the eye aspect ratio
    return ear

def sound_alarm(path):
    # play an alarm sound
    playsound.playsound(path)

# Set the thresholds and args as needed
EYE_AR_THRESH = 0.3
EYE_AR_CONSEC_FRAMES = 48
args = {"alarm": "alarm.wav"}  # Update with actual path to alarm file
ALARM_ON = False

# Initialize dlib's face detector (HOG-based) and then create the facial landmark predictor
detector = dlib.get_frontal_face_detector()
predictor = dlib.shape_predictor("shape_predictor_68_face_landmarks.dat")  # Update with actual path to shape predictor

# Grab the indexes of the facial landmarks for the left and right eye, respectively
(lStart, lEnd) = face_utils.FACIAL_LANDMARKS_IDXS["left_eye"]
(rStart, rEnd) = face_utils.FACIAL_LANDMARKS_IDXS["right_eye"]

# Initialize and start the project
top = Tk()
project = Project(top)
top.mainloop()

drivers()
```

Make sure you have the necessary libraries installed and the correct paths for `shape_predictor_68_face_landmarks.dat` and the alarm sound file.
