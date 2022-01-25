# -
The facial recognition is a very useful tool incorporated in many modern devices to detect human faces for tracking, biometric and to recognize human activities. In this project, I have used the OpenCV's Harr cascade classifiers for detecting human faces and pan/tilt servo mechanism to track the user's face using Arduino UNO.
How it Works?
Facial detection identifies and localizes human faces and ignores any background objects such as curtain, windows, trees, etc. OpenCV uses Harr cascade of classifiers where each frame of the video is passed through stages of classifiers and if the frame passes through all the classifiers, the face is present else the frame is discarded from the classifier i.e the face is not detected.

The OpenCV returns the cartesian coordinates of the image upon detection along with the height and width. From these coordinates, the center coordinates of the image can be calculated using x+width/2 and y+height/2. These coordinates are passed to the Arduino UNO using the pyserial library when the face is detected. The servo's connected to the Arduino provides a pan/tilt mechanism where the camera is connected to one of the servo. When the co-ordinates of the face is away from the center, then the servo will align by 2 degrees(increment or decrement)to bring it towards the center of the screen.

Detecting the face
I have used 'haarcascade_frontalface_default.xml' which is a pre- trained model for detecting human faces and can be downloaded from Git-Hub(here). Upon downloading, the xml file can be loaded using cv2.CascadeClassifier('haarcascade_frontalface_default.xml')

The function used for face detection is cv2.CascadeClassifier.detectMultiScale() with the 'scale factor' value as 1.1(default) and 'minNeighbour' value as 6. This returns the cartesian coordinates of the image along with the height and width. Increasing the 'minNeighbour' can improve facial detection but sacrifices in execution speeds which would lead to a delayed response from the servo. Thus, the value 6 seemed optimal.

In-order to have a precise facial recognition, a plain background would be recommended as I faced some false detection due to the curtains in the background.

Calculating the Coordinates
OpenCV returns the face coordinates in terms of pixel values. By default, the video resolution is set to 640*480. The coordinates describe the top-left pixel values(x and y) along with the height and width. I have used the center coordinates of the face for reference and can be calculated using x+width/2 and y+height/2 and can be seen as a green dot. These coordinates are sent to the arduino for moving the angle of the camera.

The square in the center of the frame in white describes the region within which the center of the face i.e the green dot must be. If it is outside the squared region when the face is moved, then the servo will align the camera to bring it inside the region.

Sending Serial data to Arduino
I found this part challenging as I tried many ways to send the coordinates sequentially to the arduino but the response was slow. After spending hours figuring it out, I began looking for similar projects online until I found this project(here). It made me aware of the Serial function Serial.parseInt() which takes integer inputs from an incoming serial of bytes(check here). My approach towards sending the serial data is similar to the one used in that project.

The python sends the center coordinates in a single string. For ex: "X100Y200", the value 100 after X represents center x-coordinate and 200 represents center-y coordinate

Arrangement of Servo
I have attached the horizontal moving servo on the shaft of the vertical moving servo in which the camera is mounted. All the attachments are made using simple rubber bands(I would not recommended it as I made use of existing material available at home).
As I am using 2 servo's for tracking, an additional 9V supply would be recommended (by means of an adapter) to the Arduino to provide sufficient current for both the servo's. In the absence of it, I have noticed some sort of vibration in them without making them move.

Libraries and Installations
This project requires pyserial and opencv libraries which I have downloaded using pip. My current python and OpenCV version is 3.8 and 4.4.0, so make sure you have a similar or a higher version. Also make sure that the XML file for face detection is saved in the same directory which contains the python script.

The python script also requires some modification(in line 9)by entering the correct COM port of your arduino before execution.


CODE

//Face Tracker using OpenCV and Arduino
//by Shubham Santosh

#include<Servo.h>

Servo x, y;
int width = 640, height = 480;  // total resolution of the video
int xpos = 90, ypos = 90;  // initial positions of both Servos
void setup() {

  Serial.begin(9600);
  x.attach(9);
  y.attach(10);
  // Serial.print(width);
  //Serial.print("\t");
  //Serial.println(height);
  x.write(xpos);
  y.write(ypos);
}
const int angle = 2;   // degree of increment or decrement

void loop() {
  if (Serial.available() > 0)
  {
    int x_mid, y_mid;
    if (Serial.read() == 'X')
    {
      x_mid = Serial.parseInt();  // read center x-coordinate
      if (Serial.read() == 'Y')
        y_mid = Serial.parseInt(); // read center y-coordinate
    }
    /* adjust the servo within the squared region if the coordinates
        is outside it
    */
    if (x_mid > width / 2 + 30)
      xpos += angle;
    if (x_mid < width / 2 - 30)
      xpos -= angle;
    if (y_mid < height / 2 + 30)
      ypos -= angle;
    if (y_mid > height / 2 - 30)
      ypos += angle;


    // if the servo degree is outside its range
    if (xpos >= 180)
      xpos = 180;
    else if (xpos <= 0)
      xpos = 0;
    if (ypos >= 180)
      ypos = 180;
    else if (ypos <= 0)
      ypos = 0;

    x.write(xpos);
    y.write(ypos);

    // used for testing
    //Serial.print("\t");
    // Serial.print(x_mid);
    // Serial.print("\t");
    // Serial.println(y_mid);
  }
}
