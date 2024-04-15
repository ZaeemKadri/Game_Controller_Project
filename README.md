# Game_Controller_Project

import cv2
import mediapipe as mp
import pyautogui

# Function to map hand gestures to keyboard keys
def map_gesture_to_key(gesture):
    if gesture == "thumbs_up":
        pyautogui.press('up')  # Move up
    elif gesture == "thumbs_down":
        pyautogui.press('down')  # Move down
    elif gesture == "thumb_right":
        pyautogui.press('right')  # Move right
    elif gesture == "thumb_left":
        pyautogui.press('left')  # Move left

# Initialize Mediapipe Hands
mp_hands = mp.solutions.hands
hands = mp_hands.Hands()

# Webcam setup
cap = cv2.VideoCapture(0)

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break

    # Convert the image to RGB
    frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)

    # Detect hands
    results = hands.process(frame_rgb)

    if results.multi_hand_landmarks:
        for hand_landmarks in results.multi_hand_landmarks:
            # Get landmarks
            landmarks = []
            for i in range(21):
                landmark = hand_landmarks.landmark[i]
                landmarks.append((landmark.x, landmark.y))

            # Check for gestures
            # For simplicity, let's assume thumb up, thumb down, index finger up, and index finger down are our gestures
            thumb = landmarks[4]
            index = landmarks[8]

            if thumb[1] < landmarks[3][1] and thumb[1] < landmarks[2][1]:
                gesture = "thumbs_up"
            elif thumb[1] > landmarks[3][1] and thumb[1] > landmarks[2][1]:
                gesture = "thumbs_down"
            elif thumb[0] < landmarks[8][0] and thumb[0] < landmarks[7][0]:  # Corrected condition here
                gesture = "thumb_right"
            elif thumb[0] > landmarks[8][0] and thumb[0] > landmarks[7][0]:
                gesture = "thumb_left"
            else:
                gesture = None

            # Map gesture to keyboard key
            if gesture:
                map_gesture_to_key(gesture)

            # Draw landmarks on hand
            for landmark in landmarks:
                x, y = int(landmark[0] * frame.shape[1]), int(landmark[1] * frame.shape[0])
                cv2.circle(frame, (x, y), 5, (255, 0, 0), -1)

    # Display the frame
    cv2.imshow("Game Controller", frame)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

# Release resources
cap.release()
cv2.destroyAllWindows()
