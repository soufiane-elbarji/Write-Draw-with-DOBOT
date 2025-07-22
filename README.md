# Write-Draw-with-DOBOT

This project allows a user to control a Dobot Magician robotic arm using voice commands. You can instruct the robot to either write a poem or draw an image on paper. The project uses Google's Gemini AI to generate the content (poems or images) and then converts it into a vector path for the robot to follow.

The process is fully automated:

1.  **Voice Input**: The system listens for a voice command like "write a poem about..." or "draw a picture of...".
2.  **AI Content Generation**: Based on the command, it uses the Gemini API to generate a unique poem or a cartoon-style image.
3.  **Path Processing**: The generated text or image is converted into a series of vector coordinates. Text is rendered into an image, and both are traced to create SVG paths.
4.  **Path Optimization**: The paths are scaled and optimized to ensure efficient movement for the Dobot arm within its drawing area.
5.  **Robotic Execution**: The final coordinates are sent to the Dobot Magician, which then draws the content onto paper.

<p align="center"\>
<img src="screenShots\Demo.gif" alt="Project Demo GIF"/\>
</p\>

## Table of Contents

  - [Requirements](https://github.com/soufiane-elbarji/Write-Draw-with-DOBOT/blob/main/README.md#requirements)
  - [Installation and Setup](https://github.com/soufiane-elbarji/Write-Draw-with-DOBOT/blob/main/README.md#installation-and-setup)
  - [Usage](https://github.com/soufiane-elbarji/Write-Draw-with-DOBOT/blob/main/README.md#usage)
  - [How It Works](https://github.com/soufiane-elbarji/Write-Draw-with-DOBOT/blob/main/README.md#how-it-works)
  - [Authors](https://github.com/soufiane-elbarji/Write-Draw-with-DOBOT/blob/main/README.md#authors)

-----

## Requirements

### Hardware

  * **Dobot Magician** robotic arm
  * A computer running **Windows**

### Software

  * **Python 3.10.10**: For running the main application logic (The version used in this project.
  * **Python 3.5.0**: Required specifically to run the `DobotDrawer.py` script, which interfaces with the `DobotDll.dll`.
  * **Google Gemini API Key**: For generating Images and Poems.
  * **Potrace**: For Transforming Image to SVG.

-----

## Installation and Setup

Follow these steps carefully to get the project running.

### 1\. Clone the Repository

Open a terminal or command prompt and clone this repository.

```bash
git clone https://github.com/your-username/Write-Draw-with-DOBOT.git
cd Write-Draw-with-DOBOT
```

### 2\. Install Python Environments

This project requires two separate versions of Python.

  * **Install Python 3.10.10**: If you don't have it, download and install it from the [official Python website](https://www.python.org/downloads/release/python-31010/). Make sure to add it to your system's PATH.

  * **Install Python 3.5.0**: This is required for the Dobot's library. Download and install it from the [official Python website](https://www.python.org/downloads/release/python-350/). Take note of the installation path.

### 3\. Install Dependencies

Install the required packages for your Python 3.10 environment using the `requirements.txt` file.

```bash
# Make sure you are using your Python 3.10 pip
pip install -r requirements.txt
```

### 4\. Configure Environment Variables

This is a critical step for the project to function correctly.

  * **`DobotDll` Path**:

    1.  Copy the full path to the `DobotDll` folder within the cloned repository directory.
    2.  Go to `Environment Variables...` and add it to the user path.

  * **`GEMINI_API_KEY`**:

    1.  In the `Environment Variables` window, under `user variables`, click `New...`.
    2.  For `Variable name`, enter `GEMINI_API_KEY`.
    3.  For `Variable value`, paste your Gemini API key.

  * **`Potrace`**:
    1.  Download `Potrace` from [The Official Website](https://potrace.sourceforge.net/#downloading).
    2.  Copy the full path of the folder.
    3.  Go to `Environment Variables...` and add it to the user path.
    4.  Click `OK` to save all changes.

**Important**: You may need to restart your computer or terminal for these environment variable changes to take effect.

### 5\. Update Python Path in `main.py`

Open the `main.py` file and update the `PYTHON35_PATH` constant to match the location of your Python 3.5 installation.

```python
# main.py
PYTHON35_PATH = "C:\\Path\\To\\Your\\Python35\\python.exe"
```

### 6\. Connect the Dobot Magician

Connect the Dobot Magician to your computer via USB and turn it on. Open the Device Manager ( `press the Windows key + R, type "devmgmt.msc" in the Run box, and then click OK or press Enter` ), find the correct COM port for the robot (e.g., `COM3`). Then, update this port in the `src/DobotDrawer.py` file.

```python
# src/DobotDrawer.py
COM_PORT = "COM3"  # Adjust to your Dobot's COM port
```

-----

## Usage

To run the project, execute the `main.py` script from your terminal in the project's root directory.

```bash
python main.py
```

1.  The program will prompt you to "Speak your request...".
2.  Use a clear voice to state your command. The command must start with "write" for poems or "draw" for images.
      * Example 1: "Write a poem about the ocean"
      * Example 2: "Draw a picture of a smiling cat"
3.  The script will display the extracted keyword and the generated content (poem or image path).
4.  It will then process the content, generate the toolpath, and automatically call the `DobotDrawer.py` script to start the drawing process.
5.  The Dobot will home itself and begin drawing. You can monitor the progress in the terminal.

-----

## How It Works

  * **Voice Recognition (`get_keyword.py`)**: Uses the `speech_recognition` library to capture and transcribe audio via Google's speech-to-text API.
  * **AI Generation (`Poemgen.py`, `Imagen.py`)**: Interfaces with the Google Gemini API to generate text or images based on the extracted keyword from the voice command.
  * **Path Generation (`get_path.py`, `get_path2.py`)**:
      * For text, it creates a bitmap (`.bmp`) image of the poem using the Pillow library.
      * For images, it uses OpenCV to detect edges and create a black-and-white outline.
      * Both bitmap types are then converted into a vector format (`.svg`) using the **Potrace** command-line tool, which is executed via a Python subprocess.
      <p align="center"\>
      <img src="screenShots\Gen_Img.png" alt="Pipeline of the Image generation to SVG paths"/\>
      </p\>
  * **Path Optimization (`optimizer.py`)**: The raw SVG paths are scaled to fit the Dobot's coordinate system. A path optimization algorithm is applied to minimize unnecessary travel moves, re-ordering and sometimes reversing strokes to create a more efficient drawing sequence.
      <p align="center"\>
      <img src="screenShots\trace_path.png" alt="Optimized path from SVG"/\>
      </p\>
  * **Dobot Control (`DobotDrawer.py`)**: This script reads the final `toolpath.txt` file, which contains a series of `MOVETO` (pen up) and `LINETO` (pen down) commands. It communicates with the `DobotDll.dll` to send these commands to the Dobot Magician's command queue for execution.

-----

## Authors

  * **[Soufiane El Barji](https://github.com/soufiane-elbarji)**
  * **[Abdellah Oubihi](https://github.com/ASTAgold)**
  * **[Nour El Houda Boulandoum](https://github.com/nour123-byte)**

-----


