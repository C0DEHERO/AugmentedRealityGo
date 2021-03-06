ARGoTrainer - Samping Chuang
2013/08/01

What?
-the purpose of this application/research is to create a system that uses augmented reality to teach the board game Go. This project is motivated by the fact that traditional ways of learning Go (textbook and computer software on monitors) is boring and discouraging for beginners. Also, learning from a computer monitor loses very important feeling that one only finds in playing with real stone and board. Therefore, we want to create a system where the user can learn go in an intuitive and entertaining environment and not losing the important physical interactions.  

How?
- The idea is that the user will wear a head mounted display (HMD) while playing a game of Go with real stones and board. During the game, the system will provide real-time assistants that teach users the strategies of Go. 
- ARGoTrainer is designed as a "Go Engine" that uses GTP (Go Text Protocol) to communicate with other Go apllications. This design allows the players to play against a physical human player or virtual opponents like Go engines (Fuego, GNUGo, etc) or online players (from KGS). The interesting thing is that the user still uses real stones even when playing against virtual opponents.


Assistants
-Currently, ARGoTrainer supports 4 modes of real-time assistants
1. Fuseki
	- Fueski means opening moves in Japanese. Like openings in chess, openings in Go is very important because player develop the most influence or territory in early games. A badly played opening moves is much harder to recover later on. Because openings is so important, the system provides assistant moves during the opening of the game. The moves are generated from Fuego using an opening book compiled from over 100,000 professional games.
2. Joseki
	- Joseki moves are studied sequences in the corner of the board. Like Fuseki, it is also very important for a Go player to know about as many Go sequences as possible to react properly in different situations. The Joseki dictionary comes from Kogo's Joseki Dictionary. The system uses pattern matching from Kombilo to search up the appropriate joseki for each of the 4 corners.
3. Go Engine Move
	-Eventually, the patterns provided by Fueski and Joseki will run out as the game progresses. However, we still want to help the user make decision during the game. So this mode provides what a Go engine (Fuego in this case) thinks is the best move.
4. Territorial Analysis
	-This is a very interesting mode that helps users to visualize the influence and territory of the stones in the current state of the board. Each point on the board is overlayed with squares of black and white. The bigger the square is, the higher probability that the color side will capture this point as a territory. The analysis is generated by Fuego's command "uct_stat_territory". 
	
System overview  (see the picture system_overview.png)
- The system is divided into 4 main categories. 
1. AR Go Controller
	The main controller of the system. It is the top layer that talks with external program with stdin/stdout using GTP (Go Text Protocol). This controller contains the board state class and spawns two parallel threads for AR Graphic Controller and AR Assistant Controller, then it sits and wait for any incoming GTP commands and react accordingly. For example, if it receives the command "play b d4," It'll send a command to the Go Board object to play a black stone at position d4. If it receives the comand "genmove b," It'll trigger a setting in AR Graphic Controller and asks it to detect any new stones placed on the board.
2. AR Graphic Controller
	- This class is responsible for both the tracking and detection of the board and rendering the virtual stones and assistances. 
	- OpenCV part: 
		- This is the very base of the application. It gets a camera image then try to locate the markers using a modified version of Aruco's marker detection code. Once the board of markers are detected, it estimates the camera pose ot obtain the rotational and translational matrices that OpenGL uses later. The more markers detected, the more accurate the estimation is. Once the Markers are detected, the system detects the Go board corners and grid points since it knows the relative position between the board and the markers in 3D space. 
		- OpenCV is also used for detecting the stones on the board. The method that we're currently using is simply thresholding the Board image obtained by undistorting the 4 corners of the Go Board from the camera's 2d image. Then we find the contours and see which grid point is each contour closest to. The problem with contours is that stones really close to each other will be identified as one big stone and be too big for our size threshould. To fix this problem, we applied some morphology operation and also drew lines between each grid lines to separate connected contours.                
		- It also detects the controller marker and calculate how much rotation has the marker made. 
	- OpenGL part:
		- This part uses the camear extrinsics from Opencv to overlay realistic stones on the board. Besides rendering the Go stones, it also overlay assistant information and error messages.
3. AR Assistant Controller
	- The class is the wrapper class to communicate with any assistant programs (such as Fuego). It creates a subprocess of the Fuego program and talks to it using GTP. The Fuseki, Go engine Move, and territorial analysis all come from Fuego. 
	- The design of the assistant is using a queue structure to hold "assistant requests." The reason is that once the user triggers a assistant, we don't want the system to hang until the assistant information is calculated and returned. Therefore, assistant controller is a separate thread running in parallel. When a user requests an assistant (by rotating the controller marker), the AR Graphic Controller calls a function in Assistant controller to push that request on the assistant queue.
	- Besides the assistant queue, there's also the "immediate queue" This queue has higher prioirty than assistant queue so requests in assistant queue is executed only when the immediate queue is empty. The immediate queue is responsible for any requests to the Fuego program that come from the AR Assistant controller (such as "play b d4" sent from AR Graphic Controller). The reason why we have this design is that there are problems when we send a command to fuego when it's in the process of calculating for one of the assistants since 
Useful Links
- GTP: http://www.lysator.liu.se/~gunnar/gtp/
- Fuego: http://fuego.sourceforge.net/
- Kombilo: http://www.u-go.net/kombilo/
- Kogo's Joseki Dictionary: http://waterfire.us/joseki.html
- FusekiLibrary: http://fusekilibrary.sourceforge.net/
- GNUGo: http://www.gnu.org/software/gnugo
- Sensei's Library: http://senseis.xmp.net/
- Boost Library: http://www.boost.org/

ArUco (augmented reality library based on OpenCV): http://www.uco.es/investiga/grupos/ava/node/26

