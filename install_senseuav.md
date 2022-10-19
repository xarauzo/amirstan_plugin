# Mega-guía para la instalación elementos de SenseUAV en AAEON

Bla bla bla

Para que los paths encajen con lo que se encuentra en el repositorio, utilizaremos el siguiente path como entorno de trabajo:

/home/aaeon/git/

## PyDeepStream

Clonamos el repositorio de 'pydeepstream2' de Gradiant.

```bash
git clone https://github.com/Gradiant/MM-PR-01394-SENSE_UAV-pydeepstream2.git
```

Creamos la imagen de docker que se utiliza en SenseAir. En este caso se hace en dos partes. La primera crea una imagen base, y a raíz de esa imagen, se crea la segunda. Esta última será la que se emplee en "pydeepstream".

```bash
cd MM-PR-01394-SENSE_UAV-pydeepstream2
docker build -f docker/Sense_base.Dockerfile -t sense_pyds:base .
docker build -f docker/Sense.Dockerfile -t sense_air_pyds:unvex .
```

```bash
sudo pip3 install pyyaml
```
> **Note**
> Hay que cambiar el campo "image_pydeepstream" en la configuración "runtime/config_files/camera_config.json" para que coincida con la imagen recien creada.

##  SenseAir

[!PREREQUISITES]
- jsonschema
```bash
sudo pip3 install jsonschema
```

Clonamos el repositorio de 'sense_air' de Gradiant, y ejecutamos el script "install.sh".

```bash
git clone https://github.com/Gradiant/MM-PR-01394-SENSE_UAV-sense_air.git
sudo ./install.sh
```
> **Note**
> Tras esto, el repo se moverá a una carpeta llamada Sense, a un nivel superior. Es posible que tengamos que cambiar ciertos paths en el código (y/o archivos de configuración) por si no encajan con nuestro path de instalación.

## Driver DJI

La instalación del driver DJI requiere de tres pasos: la instalación del Onboard-SDK (propietario de DJI), la SDK de gradiant, gradiant-uav-sdk, para el control de drones DJI, y finalmente el Driver de SenseAeronautics, que hace de interfaz entre el módulo "SenseAir" y el Dron de DJI.

### Onboard-SDK

[!PREREQUISITES]
- Install OpenSSL, FFmpeg, libUSB
```bash
sudo apt-get install libssl-dev libavcodec-dev libswresample-dev libusb-1.0-0-dev
```

Clonamos el repositorio oficial de DJI (Actualmente trabajamos en la 4.1.0):

```bash
git clone https://github.com/dji-sdk/Onboard-SDK.git
```

Para construir la OSDK hacemos lo siguiente:

```bash
cd Onboard-SDK
mkdir build
cd build
cmake .. -DADVANCED_SENSING=ON
sudo make -j7 install
```

Añadimos los permisos de escritura y lectura del UART:
```bash
sudo usermod -a -G dialout $USER
```

Sources: [DJI Developer Forum 1](https://developer.dji.com/onboard-sdk/documentation/quickstart/development-environment.html) and [DJI Developer Forum 2](https://developer.dji.com/onboard-sdk/documentation/development-workflow/sample-setup.html)

## gradiant-uav-sdk

[!PREREQUISITES]
- nlohmann/json
```bash
git clone https://github.com/nlohmann/json.git
cd json
mkdir build && cd build
cmake ..
sudo make install
```

- Catch2
```bash
git clone https://github.com/catchorg/Catch2.git
cd Catch2
mkdir build && cd build
cmake ..
sudo make install
```

- GST rtspserver
```bash
sudo apt-get install -y libgstrtspserver-1.0-dev
```

Clonamos el repositorio de xarauzo, que tiene modificaciones respecto al repositorio en bitbucket de COM.

```bash
git clone https://github.com/xarauzo/gradiant-uav-sdk.git
```
> **Note**
> Cambiamos un 'include' que hay en la librería lib_usb: /usr/local/include/linux_usb_device.hpp
	-> Lo cambioamos por "#include <libusb-1.0/libusb.h>"

Realizamos el build del SDK:
```bash
cd gradiant-uav-sdk
mkdir build && cd build
cmake ..
cmake --build .
sudo cmake --build . --target install
```

## Mavlink

Clonamos el repositorio de dplamarca:
```bash
git clone https://github.com/xarauzo/mavlink.git
```

Instalamos lo siguiente:
```bash
sudo pip3 install --user future
sudo apt install python3-lxml libxml2-utils
sudo apt-get install python3-tk
```
Añadimos el repositorio de mavlink al pythonpath:
```bash
PYTHONPATH=/home/aaeon/git/mavlink/
```

En caso de no tener las librerías dentro de ~/git/mavlink/custom_libraries, tendremos que generarlas utilizando el archivo "sense.xml" dentro de ~/git/mavlink/message_definitions/v1.0/.

Para ello usaremos la GUI, para la que necesitaremos display (accesso por ssh -X):
```bash
python3 -m mavgenerate
```
- XML: path_to_xml
- OUT: path_to_custom_libraries
- Languaje: C++
- Protocol: 2.0
- Validate: [check]
- Validate units: [check]


## Spinnaker

Download Spinnaker from: https://www.flir.com/support-center/iis/machine-vision/downloads/spinnaker-sdk-and-firmware-download/
Untar using:
```bash
tar xvfz spinnaker-<version>_arm.tar.gz
```
Run the install script:
```bash
cd spinnaker-<version>_arm/
sudo sh install_spinnaker_arm.sh
```

```bash
sudo apt install v4l2loopback-dkms
sudo apt install v4l2loopback-utils
```

Follow this: https://github.com/Gradiant/MM-PR-01394-SENSE_UAV-Camera_Drivers/tree/master/flir_driver

## DJI-Driver (Sense Aeronautics)

Clonamos el repositorio e instalamos (asegurarnos de que en el CMakeLists.txt esté bien el path al include de "mavlink.h -> /home/aaeon/git/mavlink/custom_libraries):

```bash
git clone https://github.com/Gradiant/MM-PR-01394-SENSE_UAV-DJI_driver.git
cd MM-PR-01394-SENSE_UAV-DJI_driver
mkdir build && cd build
cmake ..
sudo make -j7
```
