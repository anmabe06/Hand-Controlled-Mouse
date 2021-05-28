# Hand-Controlled-Mouse
This is a computer mouse controlled by hand gesture.
For better working, here are some recommendations:
  - Have only 1 hand visible to the camera
  - Avoid skin coloured background
  - For better performance, use a 60 fps (or higher) camera
  - Avoid very bright or dark environments
Note that by not following the listed recommendations above, in most of the case you are not preventing your camerafrom working.

Code step-by-step instruction:
1. Import libraries:
    ```
    import cv2
    import time
    import HandTrackingModule as htm
    import wx
    import numpy as np
    from pynput.mouse import Button, Controller
    
2. Import module:

3. Set initial values:
    ```
    # Helps cross-platform use
    app = wx.App(False)

    # Set hand-cursor limit
    limit = 150

    #Strength of smoothing
    smoothening = 5
    #Previus location
    prevLocationX, prevLocationY = 0, 0
    #Current location
    currentLocX, clocY = 0, 0

    # Capture video
    cap = cv2.VideoCapture(0)

    # Gets screen resolution
    wScr, hScr = wx.GetDisplaySize()
    camX, camY = int(wScr / 2), int(hScr / 2)

    # Set camera resolution
    hCam = 480
    wCam = 640
    cap.set(3, wCam)
    cap.set(4, hCam)

    # Creates mouse(as an object)
    mouse = Controller()

    # Create detector
    detector = htm.handDetector(detectionCon=0.75)

    # Mouse current location
    mouseCord = np.array([0, 0])

    #Store previous mouse location
    prevLocationX, prevLocationY = 0, 0
    #Store current mouse location
    currentLocX, cLocY = 100, 100

    # To avoid infinite clicking
    pinchFlag = 0

    #Stores screen info for better scroll movement
    screenHalf = hCam / 2
    topQuarter = hCam / 4 * 3
    bottomQuarter = hCam / 4
    
4. Make while true loop, and read each frame:
    ```
    while True:
    # Read each frame
    success, img = cap.read()
    # Return image after detecting hand
    img = detector.findHands(img)
    # Detect point positions
    lmList = detector.findPosition(img, draw=False)
    
5. Add if a hand is found condition:
    ```
    # If there is a hand
    if len(lmList) != 0:
        # Index finger position
        x1, y1 = lmList[8][1:]
        # Middle finger position
        x2, y2 = lmList[12][1:]

        # Reduce finger detection space, for better recognition
        cv2.rectangle(img, (limit, limit), (wCam - limit, hCam - limit), (255, 0, 255), 2)

        # Convert camera to screen coordinates, and apply max finger height limit
        x3 = np.interp(x1, (limit, wCam - limit), (0, wScr))
        y3 = np.interp(y1, (limit, hCam - limit), (0, hScr))
        # Smooth mouse input
        currentLocX = prevLocationX + (x3 - prevLocationX) / smoothening
        clocY = prevLocationY + (y3 - prevLocationY) / smoothening
        #Invert coordinates to mirror human
        iCurrentLocX = wScr - currentLocX
        
7. Add mouse functionalities:
    ```
    # If y coordinate of tip of index is lower than y coordinate of index knuckle
        if lmList[20][2] < lmList[17][2] and lmList[16][2] < lmList[14][2] and lmList[12][2] < lmList[10][2] and \
                lmList[8][2] < lmList[6][2]:
            if pinchFlag == 1:
                #Reset variable
                pinchFlag = 0
                # releases before clicked buttons
                mouse.release(Button.left)

            #If finger is at the top half of the screen, scroll up
            if lmList[8][2] >= screenHalf:
                # If finger is at the top quarter of the screen, scroll up faster
                if lmList[8][2] >= topQuarter:
                    mouse.scroll(0, -16)
                    print("Scroll up x2")
                # If finger is at the third quarter of the screen, scroll up faster
                elif lmList[8][2] < topQuarter:
                    mouse.scroll(0, -4)
                    print("Scroll up")
            #Elif finger is at the bottom half of the screen, scroll down
            elif lmList[8][2] < screenHalf:
                # If finger is at the second quarter of the screen, scroll up faster
                if lmList[8][2] >= bottomQuarter:
                    mouse.scroll(0, 4)
                    print("Scroll down")
                # If finger is at the first quarter of the screen, scroll up faster
                elif lmList[8][2] < bottomQuarter:
                    mouse.scroll(0, 16)
                    print("Scroll down x2")

        elif lmList[16][2] < lmList[14][2] and lmList[12][2] < lmList[10][2] and lmList[8][2] < lmList[6][2]:
            if pinchFlag == 1:
                #Reset variable
                pinchFlag = 0
                # releases before clicked buttons
                mouse.release(Button.left)

            # Click
            mouse.click(Button.right, 1)
            print("Right click")
            #Wait
            cv2.waitKey(500)

        elif lmList[12][2] < lmList[10][2] and lmList[8][2] < lmList[6][2]:
            #Calculate distance (in X axis) between both fingers
            distance = abs(lmList[12][1] - lmList[8][1])

            if distance > 30:
                if pinchFlag == 0:
                    #Update variable
                    pinchFlag = 1

                    #Click
                    mouse.press(Button.left)
                    print("Left click")

            else:
                #Click
                mouse.click(Button.left, 2)
                print("Left double click")

            # Move mouse
            mouse.position = (iCurrentLocX, clocY)

        elif lmList[8][2] < lmList[6][2]:
            if pinchFlag == 1:
                # Reset variable
                pinchFlag = 0
                # releases before clicked buttons
                mouse.release(Button.left)

            # Move mouse
            mouse.position = (iCurrentLocX, clocY)
            print("Cursor")

        #Update mouse coordinates
        prevLocationX, prevLocationY = currentLocX, clocY
        
9. Show camera:
      ```
      # Show camera
      cv2.imshow("Image", img)
      # Wait
      cv2.waitKey(1)
