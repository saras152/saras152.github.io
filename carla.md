* TOC
{:toc}

# About: 
	Current version: 0.9.11
	Uses: Unreal Engine, OpenDrive (1.4) {supports openstreetmap}, Python (3.7x), C++. 
	Server-Client Architecture
	Server responsible for simulation - sensor rendering, computation of physics, updates on the world-state and actors, etc. Run the server with a dedicated GPU. 
	Client side consists of sum of client modules - control of logic of actors on the scene and setting world conditions. CARLA API(Python or C++) layer mediates between server and client. 
	CARLA includes: 
		Traffic manager, Sensors, Recorder, Ros bridge, Autoware implementation, Open assets, Scenario runner. 
	supports CARSIM, SUMO, ROADRUNNER maps, 
	

## Quick-start to test carla. 
	run carlaue4.exe and then
	goto pythonapi/examples
	run python spawn_npc.py

### Download the version required, and the additional maps zip. The additional maps can be directly extracted to carla folder. 

### Requirements: 
	4GB GPU for server. 
	2 TCP ports (2000,2001 by default)
	64-bit OS.
	Pygame, Numpy with python3.7
	

### Some Unreal Engine command line arguments
	UE4Editor.exe -game -> to launch game using cooked content
	UE4Editor.exe -server -> run the game as a server using uncooked content. 
	MyGame.exe /Game/Maps/MyMap
	UE4Editor.exe MyGame.uproject /Game/Maps/MyMap?game=MyGameInfo -game
	UE4Editor.exe MyGame.uproject /Game/Maps/MyMap?listen -server
	MyGame.exe 127.0.0.1

# Steps for workign with carla

## 1. go to the folder and execute the following in the cmd
## 2. carlaue4.exe
## 3. carla opens in server mode by default
## 4. the spectator view opens up. 
	To move around, w,a,s,d keys can be used.
	To change viewheight, q,e can be used.
	Server is open, and waiting for conenction with the client. 
	
## 5. command line options
	-carla-rpc-port=N listen to client on port #N, streaming port is N+1
	-carla-streaming-port=N port for sending sensor data. use 0 fro random unused port. second port is N+1. 
	-quality-level={Low,Epic} to change graphics quality level 
	PythonAPI/util/config.py this script gives more command-line options
		python config.py --help # Check all the available configuration options
		
## 6. selecting GPUs on linux
	SDL_VIDEODRIVER=offscreen SDL_HINT_CUDA_DEVICE=0 ./CarlaUE4.sh -> in linux. need to check for windows. 
	Alternatively
	requirements:latest nvidia drivers, opengl, virtualgl(VGL) turbovnc2.11
	extra packages -> sudo apt install x11-xserver-utils libxrandr-dev
	configure X -> sudo nvidia-xconfig -a --use-display-device=None --virtual=1280x1024
	emulate virtual display -> sudo nohup Xorg :7 &
	run auxillary virtual x-server -> /opt/TurboVNC/bin/vncserver :8
	run on GPU 0 -> DISPLAY=:8 vglrun -d :7.0 glxinfo
	DISPLAY=:8 vglrun -d :7.<gpu_number> $CARLA_PATH/CarlaUE4/Binaries/Linux/CarlaUE4
	
## 7. running on dockers
	requirements Docker CE or Nvidia docker-2
	pull a version -> docker pull carlasim/carla:0.8.2
	run a docker -> docker run -p 2000-2002:2000-2002 --runtime=nvidia -e NVIDIA_VISIBLE_DEVICES=0 carlasim/carla:0.8.4
		NVIDIA_VISIBLE_DEVICES selects GPUs
	pass the parameters too ->docker run -p 2000-2002:2000-2002 --runtime=nvidia -e NVIDIA_VISIBLE_DEVICES=0 carlasim/carla:0.8.4 /bin/bash CarlaUE4.sh  < Your list of parameters >
		  add -world-port=<port_number> so that CARLA runs on server mode listening to the <port_number>
	
## 8. Understand timesteps

### Varaible time step is the default mode in carla. The sim time between steps is the time server takes to compute
			settings = world.get_settings()
			settings.fixed_delta_seconds = None # Set a variable time-step
			world.apply_settings(settings)
		in the fiel PythonAPI/util/config.py timestep value 0 means varaible timestep
			cd PythonAPI/util && python3 config.py --delta-seconds 0
			
### Fixed time step
			settings = world.get_settings()
			settings.fixed_delta_seconds = 0.05
			world.apply_settings(settings)
		or
			cd PythonAPI/util && python3 config.py --delta-seconds 0.05

### Physics substepping is important for calcupating physics for all objects within the timeframes
			settings = world.get_settings()
			settings.substepping = True
			settings.max_substep_delta_time = 0.01
			settings.max_substeps = 10
			world.apply_settings(settings)

### condition to be satisfied is 
			fixed_delta_seconds <= max_substep_delta_time * max_substeps
		For optimal physics simulation, substep delta below 0.016666, ideally below 0.01. 

## 9. Carla synchronization
	Default is asynchronous mode -> server runs the sim as fast as possible wihtout waiting for the client
 
### in the synchronous mode, the server waits for the client's (tick)ok message before updating the next sim step.
		in multi-client env, only one client should tick. 
			settings = world.get_settings()
			settings.synchronous_mode = True # Enables synchronous mode
			world.apply_settings(settings)
		or 
			cd PythonAPI/util && python3 config.py --no-sync # Disables synchronous mode
		If the traffic manager is enabled in ssynchronous mode, it shouls be set to synchronous mode too. 
		when using synchronous mode, use fixed time step. client runs the sim.
			dont use varaible tiem step with synchronous mode. 
		when using asychronous mode, use fixed time step. server will run as fast as possible. can sim longer periods of time 
		when using asynchronous mode, use vriable time step. this si the default carla state. 

## 10. The world and client

### the client is 
		the module that user runs to ask for info or changes in teh sim. 
		Client runs with an IP on a port. 
		communicates with the server via terminal. 
		Many clients can exist at the same time. 

### the world is
		an object representing the sim. 
			it acts as abstract layer with main methods to spawn actors, change weather, get current state of thw rodl,etc. 
			only one world per sim. 
			destryoed and new one is created when the map is changed. 

### actors
		anything that plays a role in the sim
			vehicles
			people
			sensors
			spectators
			traffic signs, lights

### Blueprints are 
			already made actor layouts that are necessary to spawn an actor. 
			models with animations and set of attributes
			attributes can be customized by users. 
			refer to blueprint library

### maps and navigation
		map is 
			object represtnging the sim world, mostly town.
				uses opendrive 1.4 to describe the roads. 
			raods, lanes, junctions are managed by Python API, accessible from the clients. 
			Used along with Waypoint class to privide vehicles with a navigation path. 
			traffic signs, traffic lights are accessible as carla.Landmark objects that have info about their opendrive def. 
			the sim generates the stops, yields, and traffic light objects when runnign using the info on opendrive file. 

### sensors and data
		sensors 
			wait for some events to happen, and gather data from sim
			call for a function defining how to manage the data. 
			is an actor attached to parent vehicle. 
				follows the vehicle around, gathering info from vehicle and surroundinsg. 
				cameras, collision detector, gnss, imu, lidar, lane invasion detector, obstacle detector, radar, rss. 

## 11. custom maps with openstreetmaps
	goto openstreetmap.org export the map region as osm file

### multi-step appraoch

#### run teh code to generate xodr file (opendrive file)
			# Read the .osm data
			f = open("path/to/osm/file", 'r') # Windows will need to encode the file in UTF-8. Read the note below. 
			osm_data = f.read()
			f.close()

			# Define the desired settings. In this case, default values.
			settings = carla.Osm2OdrSettings()
			# Convert to .xodr
			xodr_data = carla.Osm2Odr.convert(osm_data, settings)

			# save opendrive file
			f = open("path/to/output/file", 'w')
			f.write(xodr_data)
			f.close()

#### improt the map to carla
			using custom script
				vertex_distance = 2.0  # in meters
				max_road_length = 500.0 # in meters
				wall_height = 0.0      # in meters
				extra_width = 0.6      # in meters
				world = client.generate_opendrive_world(
					xodr_xml, carla.OpendriveGenerationParameters(
						vertex_distance=vertex_distance,
						max_road_length=max_road_length,
						wall_height=wall_height,
						additional_width=extra_width,
						smooth_junctions=True,
						enable_mesh_visibility=True))

### import using config.py
			python3 config.py --osm-path=/path/to/OSM/file

## 12. sim data

### start the sim by running carla
		carlaue4.exe

### map setting from one client, from the util folder
		python config.py --map Town10

### weather setting from the example folder
		python dynamic_weather.py --speed 1.0
		for custom conditions, use environment.py from util folder
			python3 environment.py --clouds 100 --rain 80 --wetness 100 --puddles 60 --wind 80 --fog 50

### set traffic
		traffic manager module handles traffic. pedestrians are handled by carla.walkeraicontroller
		python3 spawn_npc.py -n 50 -w 50 --safe // for 50 vehicles and 50 walkers

### SUMO co-simulaiton
		bidirectional. spawning in carla also spawns in sumo. 
		install, and set the environment variable SUMO_HOME, run sumo-carla synchrony script from co-simulation/sumo folder
			python3 run_synchronization.py examples/Town01.sumocfg --sumo-gui
			Sumo window opens. press 'play' to start traffic in both sims. 

### set the ego vehicle
		refer to tutorial_ego.py

#### spawns ego vehicle, soem sensors, records simulation until user finishes the script
				role_name is set to ego. 
				# --------------
				# Spawn ego vehicle
				# --------------
				ego_bp = world.get_blueprint_library().find('vehicle.tesla.model3')
				ego_bp.set_attribute('role_name','ego')
				print('\nEgo role_name is set')
				ego_color = random.choice(ego_bp.get_attribute('color').recommended_values)
				ego_bp.set_attribute('color',ego_color)
				print('\nEgo color is set')

				spawn_points = world.get_map().get_spawn_points()
				number_of_spawn_points = len(spawn_points)

				if 0 < number_of_spawn_points:
					random.shuffle(spawn_points)
					ego_transform = spawn_points[0]
					ego_vehicle = world.spawn_actor(ego_bp,ego_transform)
					print('\nEgo is spawned')
				else: 
					logging.warning('Could not found any spawn points')

#### place the spectators
				# --------------
				# Spectator on ego position
				# --------------
				spectator = world.get_spectator()
				world_snapshot = world.wait_for_tick() 
				spectator.set_transform(ego_vehicle.get_transform())
			set basic sensors
				use lib to find sensors. set specific attributes to sensor. attributes shape the data retrieved. 
				attach sensor to the vehicle. the transform is relative tothe vhielce. 
				add a listen() method. 
				try with RGB camera
					some attributes are focal_distance, shutter_speed, gamma, lens_cricle_multiplier, etc. 
					# --------------

#### # Spawn attached RGB camera
					# --------------
					cam_bp = None
					cam_bp = world.get_blueprint_library().find('sensor.camera.rgb')
					cam_bp.set_attribute("image_size_x",str(1920))
					cam_bp.set_attribute("image_size_y",str(1080))
					cam_bp.set_attribute("fov",str(105))
					cam_location = carla.Location(2,0,1)
					cam_rotation = carla.Rotation(0,180,0)
					cam_transform = carla.Transform(cam_location,cam_rotation)
					ego_cam = world.spawn_actor(cam_bp,cam_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
					ego_cam.listen(lambda image: image.save_to_disk('tutorial/output/%.6d.jpg' % image.frame))
				try some detectors
					some attributes are sensor_tick, distance, hit radius, only_dynamics, etc. 
					# --------------

#### # Add collision sensor to ego vehicle. 
					# --------------

					col_bp = world.get_blueprint_library().find('sensor.other.collision')
					col_location = carla.Location(0,0,0)
					col_rotation = carla.Rotation(0,0,0)
					col_transform = carla.Transform(col_location,col_rotation)
					ego_col = world.spawn_actor(col_bp,col_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
					def col_callback(colli):
						print("Collision detected:\n"+str(colli)+'\n')
					ego_col.listen(lambda colli: col_callback(colli))

					# --------------

#### # Add Lane invasion sensor to ego vehicle. 
					# --------------

					lane_bp = world.get_blueprint_library().find('sensor.other.lane_invasion')
					lane_location = carla.Location(0,0,0)
					lane_rotation = carla.Rotation(0,0,0)
					lane_transform = carla.Transform(lane_location,lane_rotation)
					ego_lane = world.spawn_actor(lane_bp,lane_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
					def lane_callback(lane):
						print("Lane invasion detected:\n"+str(lane)+'\n')
					ego_lane.listen(lambda lane: lane_callback(lane))

					# --------------

#### # Add Obstacle sensor to ego vehicle. 
					# --------------

					obs_bp = world.get_blueprint_library().find('sensor.other.obstacle')
					obs_bp.set_attribute("only_dynamics",str(True))
					obs_location = carla.Location(0,0,0)
					obs_rotation = carla.Rotation(0,0,0)
					obs_transform = carla.Transform(obs_location,obs_rotation)
					ego_obs = world.spawn_actor(obs_bp,obs_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
					def obs_callback(obs):
						print("Obstacle detected:\n"+str(obs)+'\n')
					ego_obs.listen(lambda obs: obs_callback(obs))
				other sensors
					# --------------

#### # Add GNSS sensor to ego vehicle. 
					# --------------

					gnss_bp = world.get_blueprint_library().find('sensor.other.gnss')
					gnss_location = carla.Location(0,0,0)
					gnss_rotation = carla.Rotation(0,0,0)
					gnss_transform = carla.Transform(gnss_location,gnss_rotation)
					gnss_bp.set_attribute("sensor_tick",str(3.0))
					ego_gnss = world.spawn_actor(gnss_bp,gnss_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
					def gnss_callback(gnss):
						print("GNSS measure:\n"+str(gnss)+'\n')
					ego_gnss.listen(lambda gnss: gnss_callback(gnss))

					# --------------

#### # Add IMU sensor to ego vehicle. 
					# --------------

					imu_bp = world.get_blueprint_library().find('sensor.other.imu')
					imu_location = carla.Location(0,0,0)
					imu_rotation = carla.Rotation(0,0,0)
					imu_transform = carla.Transform(imu_location,imu_rotation)
					imu_bp.set_attribute("sensor_tick",str(3.0))
					ego_imu = world.spawn_actor(imu_bp,imu_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
					def imu_callback(imu):
						print("IMU measure:\n"+str(imu)+'\n')
					ego_imu.listen(lambda imu: imu_callback(imu))
				advanced sensors - depth sensor
					# --------------

#### # Add a Depth camera to ego vehicle. 
					# --------------
					depth_cam = None
					depth_bp = world.get_blueprint_library().find('sensor.camera.depth')
					depth_location = carla.Location(2,0,1)
					depth_rotation = carla.Rotation(0,180,0)
					depth_transform = carla.Transform(depth_location,depth_rotation)
					depth_cam = world.spawn_actor(depth_bp,depth_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
					# This time, a color converter is applied to the image, to get the semantic segmentation view
					depth_cam.listen(lambda image: image.save_to_disk('tutorial/new_depth_output/%.6d.jpg' % image.frame,carla.ColorConverter.LogarithmicDepth))
				semantic segmentations ensor
					# --------------

#### # Add a new semantic segmentation camera to my ego
					# --------------
					sem_cam = None
					sem_bp = world.get_blueprint_library().find('sensor.camera.semantic_segmentation')
					sem_bp.set_attribute("image_size_x",str(1920))
					sem_bp.set_attribute("image_size_y",str(1080))
					sem_bp.set_attribute("fov",str(105))
					sem_location = carla.Location(2,0,1)
					sem_rotation = carla.Rotation(0,180,0)
					sem_transform = carla.Transform(sem_location,sem_rotation)
					sem_cam = world.spawn_actor(sem_bp,sem_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
					# This time, a color converter is applied to the image, to get the semantic segmentation view
					sem_cam.listen(lambda image: image.save_to_disk('tutorial/new_sem_output/%.6d.jpg' % image.frame,carla.ColorConverter.CityScapesPalette))
				Lidar ray cast sensor
					# --------------

#### # Add a new LIDAR sensor to my ego
					# --------------
					lidar_cam = None
					lidar_bp = world.get_blueprint_library().find('sensor.lidar.ray_cast')
					lidar_bp.set_attribute('channels',str(32))
					lidar_bp.set_attribute('points_per_second',str(90000))
					lidar_bp.set_attribute('rotation_frequency',str(40))
					lidar_bp.set_attribute('range',str(20))
					lidar_location = carla.Location(0,0,2)
					lidar_rotation = carla.Rotation(0,0,0)
					lidar_transform = carla.Transform(lidar_location,lidar_rotation)
					lidar_sen = world.spawn_actor(lidar_bp,lidar_transform,attach_to=ego_vehicle)
					lidar_sen.listen(lambda point_cloud: point_cloud.save_to_disk('tutorial/new_lidar_output/%.6d.ply' % point_cloud.frame))
				Lidar data visualization with meshalb. 
					install meshlab. and import the ply files into meshlab. 
				RADAR sensor
					# --------------

#### # Add a new radar sensor to my ego
					# --------------
					rad_cam = None
					rad_bp = world.get_blueprint_library().find('sensor.other.radar')
					rad_bp.set_attribute('horizontal_fov', str(35))
					rad_bp.set_attribute('vertical_fov', str(20))
					rad_bp.set_attribute('range', str(20))
					rad_location = carla.Location(x=2.0, z=1.0)
					rad_rotation = carla.Rotation(pitch=5)
					rad_transform = carla.Transform(rad_location,rad_rotation)
					rad_ego = world.spawn_actor(rad_bp,rad_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
					def rad_callback(radar_data):
						velocity_range = 7.5 # m/s
						current_rot = radar_data.transform.rotation
						for detect in radar_data:
							azi = math.degrees(detect.azimuth)
							alt = math.degrees(detect.altitude)
							# The 0.25 adjusts a bit the distance so the dots can
							# be properly seen
							fw_vec = carla.Vector3D(x=detect.depth - 0.25)
							carla.Transform(
								carla.Location(),
								carla.Rotation(
									pitch=current_rot.pitch + alt,
									yaw=current_rot.yaw + azi,
									roll=current_rot.roll)).transform(fw_vec)

							def clamp(min_v, max_v, value):
								return max(min_v, min(value, max_v))

							norm_velocity = detect.velocity / velocity_range # range [-1, 1]
							r = int(clamp(0.0, 1.0, 1.0 - norm_velocity) * 255.0)
							g = int(clamp(0.0, 1.0, 1.0 - abs(norm_velocity)) * 255.0)
							b = int(abs(clamp(- 1.0, 0.0, - 1.0 - norm_velocity)) * 255.0)
							world.debug.draw_point(
								radar_data.transform.location + fw_vec,
								size=0.075,
								life_time=0.06,
								persistent_lines=False,
								color=carla.Color(r, g, b))
					rad_ego.listen(lambda radar_data: rad_callback(radar_data))

#### No rendering mode
					good for initial simulations that can be replayed to retrieve data
					useful for situations when the simulation has extreme conditions such as dense traffic. 

#### simulate at fast pace
					disabling render quickens the sim. 
					GPU is not used, so server qoeks at full speed. 
					set fixed time step, asynchronous server, no render.
					limitation would be the inner logic of the server
					run the fixed time step, and disable rendering, asynchronous mode is already active. 
						python3 config.py --no-rendering --delta-seconds 0.05 # Never greater than 0.1s

#### manual control w/o rendering
					no_rendering_mode provides overview of sim. gives minimislictic arial view with pygame. 
					can be used aling with manual-control to generate a route, rrecord, and play back to gather data. 
						python3 manual_control.py
						python3 no_rendering_mode.py --no-rendering

### record and retrieve data

#### start recording into CarlaUE4/Saved
			# --------------
			# Start recording
			# --------------
			client.start_recorder('~/tutorial/recorder/recording01.log')

#### enable auto pilot
			# --------------
			# Capture data
			# --------------
			ego_vehicle.set_autopilot(True)
			print('\nEgo autopilot enabled')
			while True:
				world_snapshot = world.wait_for_tick()

#### manual control
			run manual_control.py in one client, and recorder in anotehr one. 
			to avoid rendering and save computational cost, enable no-rendering mode. no_rendering_mode.py creates a simple arial view. 

#### stop recording
			client.stop_recorder()

#### examine the recording
			./CarlaUE4.sh
			python3 tuto_replay.py
				# --------------
				# Reenact a fragment of the recording
				# --------------
				client.replay_file("~/tutorial/recorder/recording01.log",45,10,0)

#### query the events
				# --------------
				# Query the recording
				# --------------
				# Show only the most important events in the recording.  
				print(client.show_recorder_file_info("~/tutorial/recorder/recording01.log",False))
				# Show actors not moving 1 meter in 10 seconds.  
				print(client.show_recorder_actors_blocked("~/tutorial/recorder/recording01.log",10,1))
				# Filter collisions between vehicles 'v' and 'a' any other type of actor.  
				print(client.show_recorder_collisions("~/tutorial/recorder/recording01.log",'v','a'))

## 13. tutorial_ego.py
		import glob
		import os
		import sys
		import time

		try:
			sys.path.append(glob.glob('../carla/dist/carla-*%d.%d-%s.egg' % (
				sys.version_info.major,
				sys.version_info.minor,
				'win-amd64' if os.name == 'nt' else 'linux-x86_64'))[0])
		except IndexError:
			pass

		import carla

		import argparse
		import logging
		import random


		def main():
			argparser = argparse.ArgumentParser(
				description=__doc__)
			argparser.add_argument(
				'--host',
				metavar='H',
				default='127.0.0.1',
				help='IP of the host server (default: 127.0.0.1)')
			argparser.add_argument(
				'-p', '--port',
				metavar='P',
				default=2000,
				type=int,
				help='TCP port to listen to (default: 2000)')
			args = argparser.parse_args()

			logging.basicConfig(format='%(levelname)s: %(message)s', level=logging.INFO)

			client = carla.Client(args.host, args.port)
			client.set_timeout(10.0)

			try:

				world = client.get_world()
				ego_vehicle = None
				ego_cam = None
				ego_col = None
				ego_lane = None
				ego_obs = None
				ego_gnss = None
				ego_imu = None

				# --------------
				# Start recording
				# --------------
				"""
				client.start_recorder('~/tutorial/recorder/recording01.log')
				"""

				# --------------
				# Spawn ego vehicle
				# --------------
				"""
				ego_bp = world.get_blueprint_library().find('vehicle.tesla.model3')
				ego_bp.set_attribute('role_name','ego')
				print('\nEgo role_name is set')
				ego_color = random.choice(ego_bp.get_attribute('color').recommended_values)
				ego_bp.set_attribute('color',ego_color)
				print('\nEgo color is set')

				spawn_points = world.get_map().get_spawn_points()
				number_of_spawn_points = len(spawn_points)

				if 0 < number_of_spawn_points:
					random.shuffle(spawn_points)
					ego_transform = spawn_points[0]
					ego_vehicle = world.spawn_actor(ego_bp,ego_transform)
					print('\nEgo is spawned')
				else: 
					logging.warning('Could not found any spawn points')
				"""

				# --------------
				# Add a RGB camera sensor to ego vehicle. 
				# --------------
				"""
				cam_bp = None
				cam_bp = world.get_blueprint_library().find('sensor.camera.rgb')
				cam_bp.set_attribute("image_size_x",str(1920))
				cam_bp.set_attribute("image_size_y",str(1080))
				cam_bp.set_attribute("fov",str(105))
				cam_location = carla.Location(2,0,1)
				cam_rotation = carla.Rotation(0,180,0)
				cam_transform = carla.Transform(cam_location,cam_rotation)
				ego_cam = world.spawn_actor(cam_bp,cam_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
				ego_cam.listen(lambda image: image.save_to_disk('~/tutorial/output/%.6d.jpg' % image.frame))
				"""

				# --------------
				# Add collision sensor to ego vehicle. 
				# --------------
				"""
				col_bp = world.get_blueprint_library().find('sensor.other.collision')
				col_location = carla.Location(0,0,0)
				col_rotation = carla.Rotation(0,0,0)
				col_transform = carla.Transform(col_location,col_rotation)
				ego_col = world.spawn_actor(col_bp,col_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
				def col_callback(colli):
					print("Collision detected:\n"+str(colli)+'\n')
				ego_col.listen(lambda colli: col_callback(colli))
				"""

				# --------------
				# Add Lane invasion sensor to ego vehicle. 
				# --------------
				"""
				lane_bp = world.get_blueprint_library().find('sensor.other.lane_invasion')
				lane_location = carla.Location(0,0,0)
				lane_rotation = carla.Rotation(0,0,0)
				lane_transform = carla.Transform(lane_location,lane_rotation)
				ego_lane = world.spawn_actor(lane_bp,lane_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
				def lane_callback(lane):
					print("Lane invasion detected:\n"+str(lane)+'\n')
				ego_lane.listen(lambda lane: lane_callback(lane))
				"""

				# --------------
				# Add Obstacle sensor to ego vehicle. 
				# --------------
				"""
				obs_bp = world.get_blueprint_library().find('sensor.other.obstacle')
				obs_bp.set_attribute("only_dynamics",str(True))
				obs_location = carla.Location(0,0,0)
				obs_rotation = carla.Rotation(0,0,0)
				obs_transform = carla.Transform(obs_location,obs_rotation)
				ego_obs = world.spawn_actor(obs_bp,obs_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
				def obs_callback(obs):
					print("Obstacle detected:\n"+str(obs)+'\n')
				ego_obs.listen(lambda obs: obs_callback(obs))
				"""

				# --------------
				# Add GNSS sensor to ego vehicle. 
				# --------------
				"""
				gnss_bp = world.get_blueprint_library().find('sensor.other.gnss')
				gnss_location = carla.Location(0,0,0)
				gnss_rotation = carla.Rotation(0,0,0)
				gnss_transform = carla.Transform(gnss_location,gnss_rotation)
				gnss_bp.set_attribute("sensor_tick",str(3.0))
				ego_gnss = world.spawn_actor(gnss_bp,gnss_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
				def gnss_callback(gnss):
					print("GNSS measure:\n"+str(gnss)+'\n')
				ego_gnss.listen(lambda gnss: gnss_callback(gnss))
				"""

				# --------------
				# Add IMU sensor to ego vehicle. 
				# --------------
				"""
				imu_bp = world.get_blueprint_library().find('sensor.other.imu')
				imu_location = carla.Location(0,0,0)
				imu_rotation = carla.Rotation(0,0,0)
				imu_transform = carla.Transform(imu_location,imu_rotation)
				imu_bp.set_attribute("sensor_tick",str(3.0))
				ego_imu = world.spawn_actor(imu_bp,imu_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
				def imu_callback(imu):
					print("IMU measure:\n"+str(imu)+'\n')
				ego_imu.listen(lambda imu: imu_callback(imu))
				"""

				# --------------
				# Place spectator on ego spawning
				# --------------
				"""
				spectator = world.get_spectator()
				world_snapshot = world.wait_for_tick() 
				spectator.set_transform(ego_vehicle.get_transform())
				"""

				# --------------
				# Enable autopilot for ego vehicle
				# --------------
				"""
				ego_vehicle.set_autopilot(True)
				"""

				# --------------
				# Game loop. Prevents the script from finishing.
				# --------------
				while True:
					world_snapshot = world.wait_for_tick()

			finally:
				# --------------
				# Stop recording and destroy actors
				# --------------
				client.stop_recorder()
				if ego_vehicle is not None:
					if ego_cam is not None:
						ego_cam.stop()
						ego_cam.destroy()
					if ego_col is not None:
						ego_col.stop()
						ego_col.destroy()
					if ego_lane is not None:
						ego_lane.stop()
						ego_lane.destroy()
					if ego_obs is not None:
						ego_obs.stop()
						ego_obs.destroy()
					if ego_gnss is not None:
						ego_gnss.stop()
						ego_gnss.destroy()
					if ego_imu is not None:
						ego_imu.stop()
						ego_imu.destroy()
					ego_vehicle.destroy()

		if __name__ == '__main__':

			try:
				main()
			except KeyboardInterrupt:
				pass
			finally:
				print('\nDone with tutorial_ego.')
	

## 14. tutorial_replay.py
		import glob
		import os
		import sys
		import time
		import math
		import weakref

		try:
			sys.path.append(glob.glob('../carla/dist/carla-*%d.%d-%s.egg' % (
				sys.version_info.major,
				sys.version_info.minor,
				'win-amd64' if os.name == 'nt' else 'linux-x86_64'))[0])
		except IndexError:
			pass

		import carla

		import argparse
		import logging
		import random

		def main():
			client = carla.Client('127.0.0.1', 2000)
			client.set_timeout(10.0)

			try:

				world = client.get_world() 
				ego_vehicle = None
				ego_cam = None
				depth_cam = None
				depth_cam02 = None
				sem_cam = None
				rad_ego = None
				lidar_sen = None

				# --------------
				# Query the recording
				# --------------
				"""
				# Show the most important events in the recording.  
				print(client.show_recorder_file_info("~/tutorial/recorder/recording05.log",False))
				# Show actors not moving 1 meter in 10 seconds.  
				#print(client.show_recorder_actors_blocked("~/tutorial/recorder/recording04.log",10,1))
				# Show collisions between any type of actor.  
				#print(client.show_recorder_collisions("~/tutorial/recorder/recording04.log",'v','a'))
				"""

				# --------------
				# Reenact a fragment of the recording
				# --------------
				"""
				client.replay_file("~/tutorial/recorder/recording03.log",0,30,0)
				"""

				# --------------
				# Set playback simulation conditions
				# --------------
				"""
				ego_vehicle = world.get_actor(322) #Store the ID from the simulation or query the recording to find out
				"""

				# --------------
				# Place spectator on ego spawning
				# --------------
				"""
				spectator = world.get_spectator()
				world_snapshot = world.wait_for_tick() 
				spectator.set_transform(ego_vehicle.get_transform())
				"""

				# --------------
				# Change weather conditions
				# --------------
				"""
				weather = world.get_weather()
				weather.sun_altitude_angle = -30
				weather.fog_density = 65
				weather.fog_distance = 10
				world.set_weather(weather)
				"""

				# --------------
				# Add a RGB camera to ego vehicle.
				# --------------
				"""
				cam_bp = None
				cam_bp = world.get_blueprint_library().find('sensor.camera.rgb')
				cam_location = carla.Location(2,0,1)
				cam_rotation = carla.Rotation(0,180,0)
				cam_transform = carla.Transform(cam_location,cam_rotation)
				cam_bp.set_attribute("image_size_x",str(1920))
				cam_bp.set_attribute("image_size_y",str(1080))
				cam_bp.set_attribute("fov",str(105))
				ego_cam = world.spawn_actor(cam_bp,cam_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
				ego_cam.listen(lambda image: image.save_to_disk('~/tutorial/new_rgb_output/%.6d.jpg' % image.frame))
				"""

				# --------------
				# Add a Logarithmic Depth camera to ego vehicle. 
				# --------------
				"""
				depth_cam = None
				depth_bp = world.get_blueprint_library().find('sensor.camera.depth')
				depth_bp.set_attribute("image_size_x",str(1920))
				depth_bp.set_attribute("image_size_y",str(1080))
				depth_bp.set_attribute("fov",str(105))
				depth_location = carla.Location(2,0,1)
				depth_rotation = carla.Rotation(0,180,0)
				depth_transform = carla.Transform(depth_location,depth_rotation)
				depth_cam = world.spawn_actor(depth_bp,depth_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
				# This time, a color converter is applied to the image, to get the semantic segmentation view
				depth_cam.listen(lambda image: image.save_to_disk('~/tutorial/de_log/%.6d.jpg' % image.frame,carla.ColorConverter.LogarithmicDepth))
				"""
				# --------------
				# Add a Depth camera to ego vehicle. 
				# --------------
				"""
				depth_cam02 = None
				depth_bp02 = world.get_blueprint_library().find('sensor.camera.depth')
				depth_bp02.set_attribute("image_size_x",str(1920))
				depth_bp02.set_attribute("image_size_y",str(1080))
				depth_bp02.set_attribute("fov",str(105))
				depth_location02 = carla.Location(2,0,1)
				depth_rotation02 = carla.Rotation(0,180,0)
				depth_transform02 = carla.Transform(depth_location02,depth_rotation02)
				depth_cam02 = world.spawn_actor(depth_bp02,depth_transform02,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
				# This time, a color converter is applied to the image, to get the semantic segmentation view
				depth_cam02.listen(lambda image: image.save_to_disk('~/tutorial/de/%.6d.jpg' % image.frame,carla.ColorConverter.Depth))
				"""

				# --------------
				# Add a new semantic segmentation camera to ego vehicle
				# --------------
				"""
				sem_cam = None
				sem_bp = world.get_blueprint_library().find('sensor.camera.semantic_segmentation')
				sem_bp.set_attribute("image_size_x",str(1920))
				sem_bp.set_attribute("image_size_y",str(1080))
				sem_bp.set_attribute("fov",str(105))
				sem_location = carla.Location(2,0,1)
				sem_rotation = carla.Rotation(0,180,0)
				sem_transform = carla.Transform(sem_location,sem_rotation)
				sem_cam = world.spawn_actor(sem_bp,sem_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
				# This time, a color converter is applied to the image, to get the semantic segmentation view
				sem_cam.listen(lambda image: image.save_to_disk('~/tutorial/new_sem_output/%.6d.jpg' % image.frame,carla.ColorConverter.CityScapesPalette))
				"""

				# --------------
				# Add a new radar sensor to ego vehicle
				# --------------
				"""
				rad_cam = None
				rad_bp = world.get_blueprint_library().find('sensor.other.radar')
				rad_bp.set_attribute('horizontal_fov', str(35))
				rad_bp.set_attribute('vertical_fov', str(20))
				rad_bp.set_attribute('range', str(20))
				rad_location = carla.Location(x=2.8, z=1.0)
				rad_rotation = carla.Rotation(pitch=5)
				rad_transform = carla.Transform(rad_location,rad_rotation)
				rad_ego = world.spawn_actor(rad_bp,rad_transform,attach_to=ego_vehicle, attachment_type=carla.AttachmentType.Rigid)
				def rad_callback(radar_data):
					velocity_range = 7.5 # m/s
					current_rot = radar_data.transform.rotation
					for detect in radar_data:
						azi = math.degrees(detect.azimuth)
						alt = math.degrees(detect.altitude)
						# The 0.25 adjusts a bit the distance so the dots can
						# be properly seen
						fw_vec = carla.Vector3D(x=detect.depth - 0.25)
						carla.Transform(
							carla.Location(),
							carla.Rotation(
								pitch=current_rot.pitch + alt,
								yaw=current_rot.yaw + azi,
								roll=current_rot.roll)).transform(fw_vec)

						def clamp(min_v, max_v, value):
							return max(min_v, min(value, max_v))

						norm_velocity = detect.velocity / velocity_range # range [-1, 1]
						r = int(clamp(0.0, 1.0, 1.0 - norm_velocity) * 255.0)
						g = int(clamp(0.0, 1.0, 1.0 - abs(norm_velocity)) * 255.0)
						b = int(abs(clamp(- 1.0, 0.0, - 1.0 - norm_velocity)) * 255.0)
						world.debug.draw_point(
							radar_data.transform.location + fw_vec,
							size=0.075,
							life_time=0.06,
							persistent_lines=False,
							color=carla.Color(r, g, b))
				rad_ego.listen(lambda radar_data: rad_callback(radar_data))
				"""

				# --------------
				# Add a new LIDAR sensor to ego vehicle
				# --------------
				"""
				lidar_cam = None
				lidar_bp = world.get_blueprint_library().find('sensor.lidar.ray_cast')
				lidar_bp.set_attribute('channels',str(32))
				lidar_bp.set_attribute('points_per_second',str(90000))
				lidar_bp.set_attribute('rotation_frequency',str(40))
				lidar_bp.set_attribute('range',str(20))
				lidar_location = carla.Location(0,0,2)
				lidar_rotation = carla.Rotation(0,0,0)
				lidar_transform = carla.Transform(lidar_location,lidar_rotation)
				lidar_sen = world.spawn_actor(lidar_bp,lidar_transform,attach_to=ego_vehicle,attachment_type=carla.AttachmentType.Rigid)
				lidar_sen.listen(lambda point_cloud: point_cloud.save_to_disk('/home/adas/Desktop/tutorial/new_lidar_output/%.6d.ply' % point_cloud.frame))
				"""

				# --------------
				# Game loop. Prevents the script from finishing.
				# --------------
				while True:
					world_snapshot = world.wait_for_tick()

			finally:
				# --------------
				# Destroy actors
				# --------------
				if ego_vehicle is not None:
					if ego_cam is not None:
						ego_cam.stop()
						ego_cam.destroy()
					if depth_cam is not None:
						depth_cam.stop()
						depth_cam.destroy()
					if sem_cam is not None:
						sem_cam.stop()
						sem_cam.destroy()
					if rad_ego is not None:
						rad_ego.stop()
						rad_ego.destroy()
					if lidar_sen is not None:
						lidar_sen.stop()
						lidar_sen.destroy()
					ego_vehicle.destroy()
				print('\nNothing to be done.')


		if __name__ == '__main__':

			try:
				main()
			except KeyboardInterrupt:
				pass
			finally:
				print('\nDone with tutorial_replay.')

## 15. create custom vehicles
