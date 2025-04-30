# my_simple_arm_description

Este repositorio contiene la descripción URDF y el launch file para simular un brazo robótico simple en ROS 2 y Gazebo.

## Contenido

- `urdf/test_arm.urdf` – Descripción del brazo en formato URDF.
- `launch/complete_arm_spawn.launch.py` – Launch file para iniciar Gazebo, publicar la descripción del robot y spawnear la entidad.

## Requisitos previos

- **ROS 2** (Foxy, Galactic o Humble) instalado y configurado.
- **gazebo_ros** instalado:
  ```bash
  sudo apt install ros-$(ros2 distro show)/gazebo-ros-pkgs ros-$(ros2 distro show)/gazebo-ros-control
  ```
- Workspace de ROS 2 creado y con el entorno fuente:
  ```bash
  source ~/ros2_ws/install/setup.bash
  ```
- Paquete `my_simple_arm_description` ubicado en `~/ros2_ws/src/`.

## Estructura de directorios

```bash
ros2_ws/
└── src/
    └── my_simple_arm_description/
        ├── launch/
        │   └── complete_arm_spawn.launch.py
        └── urdf/
            └── test_arm.urdf
```

Si aún no has creado el paquete, hazlo con:
```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_cmake my_simple_arm_description
```

## Paso 1: Crear la descripción URDF

```bash
mkdir -p ~/ros2_ws/src/my_simple_arm_description/urdf
cat > ~/ros2_ws/src/my_simple_arm_description/urdf/test_arm.urdf << 'EOF'
<?xml version="1.0"?>
<robot name="test_arm">
  <!-- Base cilíndrica azul -->
  <link name="base_link">
    <visual>
      <geometry><cylinder length="0.1" radius="0.2"/></geometry>
      <material name="blue"><color rgba="0 0 1 1"/></material>
    </visual>
    <collision>
      <geometry><cylinder length="0.1" radius="0.2"/></geometry>
    </collision>
    <inertial>
      <mass value="1.0"/>
      <inertia ixx="0.1" ixy="0" ixz="0" iyy="0.1" iyz="0" izz="0.1"/>
    </inertial>
  </link>

  <!-- Eslabón del brazo rojo -->
  <link name="arm_link">
    <visual>
      <geometry><box size="0.1 0.1 0.5"/></geometry>
      <origin xyz="0 0 0.25" rpy="0 0 0"/>
      <material name="red"><color rgba="1 0 0 1"/></material>
    </visual>
    <collision>
      <geometry><box size="0.1 0.1 0.5"/></geometry>
      <origin xyz="0 0 0.25" rpy="0 0 0"/>
    </collision>
    <inertial>
      <mass value="0.5"/>
      <origin xyz="0 0 0.25" rpy="0 0 0"/>
      <inertia ixx="0.01" ixy="0" ixz="0" iyy="0.01" iyz="0" izz="0.01"/>
    </inertial>
  </link>

  <!-- Unión base→brazo -->
  <joint name="base_to_arm" type="revolute">
    <parent link="base_link"/>
    <child link="arm_link"/>
    <origin xyz="0 0 0.05" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>
    <limit lower="-1.57" upper="1.57" velocity="1.0" effort="100"/>
  </joint>

  <!-- Efector final verde -->
  <link name="end_effector">
    <visual>
      <geometry><sphere radius="0.1"/></geometry>
      <material name="green"><color rgba="0 1 0 1"/></material>
    </visual>
    <collision>
      <geometry><sphere radius="0.1"/></geometry>
    </collision>
    <inertial>
      <mass value="0.1"/>
      <inertia ixx="0.001" ixy="0" ixz="0" iyy="0.001" iyz="0" izz="0.001"/>
    </inertial>
  </link>

  <!-- Unión brazo→efector -->
  <joint name="arm_to_end_effector" type="revolute">
    <parent link="arm_link"/>
    <child link="end_effector"/>
    <origin xyz="0 0 0.5" rpy="0 0 0"/>
    <axis xyz="1 0 0"/>
    <limit lower="-1.57" upper="1.57" velocity="1.0" effort="100"/>
  </joint>

  <!-- Colores en Gazebo -->
  <gazebo reference="base_link"><material>Gazebo/Blue</material></gazebo>
  <gazebo reference="arm_link"><material>Gazebo/Red</material></gazebo>
  <gazebo reference="end_effector"><material>Gazebo/Green</material></gazebo>
</robot>
EOF
```

## Paso 2: Crear el launch file

```bash
mkdir -p ~/ros2_ws/src/my_simple_arm_description/launch
cat > ~/ros2_ws/src/my_simple_arm_description/launch/complete_arm_spawn.launch.py << 'EOF'
import os
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch_ros.actions import Node
from ament_index_python.packages import get_package_share_directory

def generate_launch_description():
    pkg_gazebo_ros = get_package_share_directory('gazebo_ros')
    urdf_file_path = os.path.join(
        os.path.dirname(os.path.dirname(os.path.realpath(__file__))),
        'urdf', 'test_arm.urdf'
    )
    gazebo = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(
            os.path.join(pkg_gazebo_ros, 'launch', 'gazebo.launch.py')
        )
    )
    with open(urdf_file_path, 'r') as urdf_file:
        robot_description = urdf_file.read()
    robot_state_publisher = Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        parameters=[{'robot_description': robot_description}],
        output='screen'
    )
    spawn_entity = Node(
        package='gazebo_ros',
        executable='spawn_entity.py',
        arguments=['-entity', 'test_arm', '-file', urdf_file_path, '-x', '0.0', '-y', '0.0', '-z', '0.5'],
        output='screen'
    )
    return LaunchDescription([gazebo, robot_state_publisher, spawn_entity])
EOF
```

## Paso 3: Permisos de ejecución

```bash
chmod +x ~/ros2_ws/src/my_simple_arm_description/launch/complete_arm_spawn.launch.py
```

## Paso 4: Compilar el paquete

```bash
cd ~/ros2_ws
colcon build --packages-select my_simple_arm_description
source install/setup.bash
```

## Paso 5: Lanzar la simulación

```bash
ros2 launch my_simple_arm_description complete_arm_spawn.launch.py
```

- Si Gazebo estaba abierto, ciérralo antes (Ctrl+C).  
- Verás el brazo en Gazebo con colores azul, rojo y verde.

## Solución de problemas comunes

| Síntoma                     | Causa posible                       | Solución                                        |
|-----------------------------|-------------------------------------|-------------------------------------------------|
| URDF no encontrado          | Ruta o nombre de archivo incorrecto | Verifica `urdf/test_arm.urdf`                   |
| Robot no aparece en Gazebo  | SpawnEntity fallido                 | Revisa errores en la consola                    |
| Joints sin movimiento       | Faltan plugins de gazebo_ros        | Instala `gazebo_ros_pkgs` y `.ros2 control`     |
| Colores sin efecto en Gazebo| Tags `<gazebo>` omitidos            | Revisa `<gazebo>` en URDF y reinicia Gazebo     |

## Próximos pasos

- Añadir más enlaces y articulaciones.
- Integrar controladores con ROS 2 Control.
- Incorporar sensores o cámaras.
- Ajustar dinámicas e inercias para mayor realismo.

---

¡Listo! Este README.md te guiará para desplegar tu brazo simple en ROS 2 y Gazebo.

