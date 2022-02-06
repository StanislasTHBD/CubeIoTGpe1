# Raspberry Pi Zéro 

![alt text](https://github.com/StanislasTHBD/CubeIoTGpe1/blob/StanislasTHBD-patch-1/Raspberry_Pi_Zero.png)

# Étape 1 : Installation OS
- Installer os « Raspberry Pi OS Lite » sur carte SD 

		Lien : https://downloads.raspberrypi.org/raspios_lite_armhf/images/raspios_lite_armhf-2022-01-28/2022-01-28-raspios-bullseye-armhf-lite.zip
		
- Configuration du Raspberry Pi 
Ajouter un fichier wpa_supplicant.conf =>

	    • ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
		update_config=1
		country=FR
		network={
			ssid="reseau_nom"
			psk="reseau_mdp"
			scan_ssid=1
			key_mgmt=WPA-PSK
			}
- Ajouter un fichier ssh (sans extension) vide

# Étape 2 : Branchement & connexion avec le Raspberry Pi
- Brancher le raspberry et le pinger avec la commande pour trouver l’adresse IP :

	    ping raspberrypi.local
	    
- Connexion ssh => pi@adresse_ip

	    ssh pi@192.168.43.60
 	    Mdp : raspberry

# Étape 3 : Installation de l'environnement du Raspberry

	sudo apt-get update && apt-get upgrade
	sudo apt-get install php php-mysql apache2 mariadb-server
	mysql_secure_installation
	sudo apt-get install phpmyadmin

# Étape 4 : Page de connexion à phpMyAdmin
- Taper dans URL:

		192.168.43.60/phpmyadmin

- Pour se connecter:

		user : root
		mdp : root 

# ESP8266

![alt text](https://github.com/StanislasTHBD/CubeIoTGpe1/blob/StanislasTHBD-patch-1/ESP8266.png)

# Schéma du câblage : ESP8266

![alt text](https://github.com/StanislasTHBD/CubeIoTGpe1/blob/StanislasTHBD-patch-1/Montage.png)

# Étape 1 : Installation & Configuration ESP8266 
Pour effacer le flash du ESP8266 :

	python3 -m esptool erase_flash
    
Pour mettre le flash sur l’ ESP8266 :

	python3 -m esptool write_flash –flash_size=1MB 0 ./Téléchargements/esp8266-lm-20220117-v1.18.bin

# Étape 2 : Programme en micropython
- Utilisation du Logiciel "Thonny" pour l'ESP8266

- Fichier de test: "connexion.py" permet de se connecter au partage de connexion via téléphone

		try:
		  import usocket as socket
		except:
		  import socket

		from machine import Pin
		import network

		import esp
		esp.osdebug(None)

		import gc
		gc.collect()

		ssid = 'reseau_nom’
		password = 'reseau_mdp'

		station = network.WLAN(network.STA_IF)

		station.active(True)
		station.connect(ssid, password)

		while station.isconnected() == False:
		  pass

		print('Connection successful')
		print(station.ifconfig()) 

- Library Capteur de température & humidité: fichier LIB "si7021.py"

		'''This module implements a driver for the Si7021 humidity and temperature
		sensor.
		Datasheet:
		https://www.silabs.com/Support%20Documents%2FTechnicalDocs%2FSi7021-A20.pdf
		'''

		from time import sleep

		class CRCError(Exception):
		    'Data failed a CRC check.'
		    pass


		class Si7021(object):
		    'Driver for the Si7021 temperature sensor.'
		    SI7021_DEFAULT_ADDRESS = 0x40
		    SI7021_MEASTEMP_NOHOLD_CMD = bytearray([0xF3])
		    SI7021_MEASRH_NOHOLD_CMD = bytearray([0xF5])
		    SI7021_RESET_CMD = bytearray([0xFE])
		    SI7021_ID1_CMD = bytearray([0xFA, 0x0F])
		    SI7021_ID2_CMD = bytearray([0xFC, 0xC9])
		    I2C_WAIT_TIME = 0.025


		    def __init__(self, i2c, address=SI7021_DEFAULT_ADDRESS):
		        'Initialize an Si7021 sensor object.'
		        self.i2c = i2c
		        self.address = address
		        self.serial, self.identifier = self._get_device_info()


		    @property
		    def temperature(self):
		        'Return the temperature in Celcius.'
		        temperature = self._get_data(self.SI7021_MEASTEMP_NOHOLD_CMD)
		        celcius = temperature * 175.72 / 65536 - 46.85
		        return celcius


		    @temperature.setter
		    def temperature(self, value):
		        raise AttributeError('can\'t set attribute')


		    @property
		    def relative_humidity(self):
		        'Return the relative humidity as a percentage. i.e. 35.59927'
		        relative_humidity = self._get_data(self.SI7021_MEASRH_NOHOLD_CMD)
		        relative_humidity = relative_humidity * 125 / 65536 - 6
		        return relative_humidity


		    @relative_humidity.setter
		    def relative_humidity(self, value):
		        raise AttributeError('can\'t set attribute')


		    def reset(self):
		        'Reset the sensor.'
		        self.i2c.writeto(self.address, self.SI7021_RESET_CMD)
		        sleep(self.I2C_WAIT_TIME)


		    def _get_data(self, command):
		        'Retrieve data from the sensor and verify it with a CRC check.'
		        data = bytearray(3)
		        self.i2c.writeto(self.address, command)
		        sleep(self.I2C_WAIT_TIME)

		        self.i2c.readfrom_into(self.address, data)
		        value = self._convert_to_integer(data[:2])

		        verified = self._verify_checksum(data)
		        if not verified:
		            raise CRCError('Data read off i2c bus failed CRC check.',
		                           data[:2],
		                           data[-1])
		        return value


		    def _get_device_info(self):
		        '''Get the serial number and the sensor identifier. The identifier is
		        part of the bytes returned for the serial number.
		        '''
		        # Serial 1st half
		        self.i2c.writeto(self.address, self.SI7021_ID1_CMD)
		        id1 = bytearray(8)
		        sleep(self.I2C_WAIT_TIME)
		        self.i2c.readfrom_into(self.address, id1)

		        # Serial 2nd half
		        self.i2c.writeto(self.address, self.SI7021_ID2_CMD)
		        id2 = bytearray(6)
		        sleep(self.I2C_WAIT_TIME)
		        self.i2c.readfrom_into(self.address, id2)

		        combined_id = bytearray([id1[0], id1[2], id1[4], id1[6],
		                                 id2[0], id2[1], id2[3], id2[4]])

		        serial = self._convert_to_integer(combined_id)
		        identifier = self._get_device_identifier(id2[0])

		        return serial, identifier

		    def _convert_to_integer(self, bytes_to_convert):
		        'Use bitwise operators to convert the bytes into integers.'
		        integer = None
		        for chunk in bytes_to_convert:
		            if not integer:
		                integer = chunk
		            else:
		                integer = integer << 8
		                integer = integer | chunk
		        return integer


		    def _get_device_identifier(self, identifier_byte):
		        '''Convert the identifier byte to a device identifier. Values are based
		        on the information from page 24 of the datasheet.
		        '''
		        if identifier_byte == 0x00 or identifier_byte == 0xFF:
		            return 'engineering sample'
		        elif identifier_byte == 0x0D:
		            return 'Si7013'
		        elif identifier_byte == 0x14:
		            return 'Si7020'
		        elif identifier_byte == 0x15:
		            return 'Si7021'
		        else:
		            return 'unknown'


		    def _verify_checksum(self, data):
		        ''''Verify the checksum using the polynomial from page 19 of the
		        datasheet.
		        x8 + x5 + x4 + 1 = 0x131 = 0b100110001
		        Valid Example:
		        byte1: 0x67 [01100111]
		        byte2: 0x8c [10001100]
		        byte3: 0xfc [11111100] (CRC byte)
		        '''
		        crc = 0
		        values = data[:2]
		        checksum = int(data[-1])
		        for value in values:
		            crc = crc ^ value
		            for _ in range(8, 0, -1):
		                if crc & 0x80: #10000000
		                    crc <<= 1
		                    crc ^= 0x131 #100110001
		                else:
		                    crc <<= 1
		        if crc != checksum:
		            return False
		        else:
		            return True

		def convert_celcius_to_fahrenheit(celcius):
		    'Convert a Celcius measurement into a Fahrenheit measurement.'
		    return celcius * 1.8 + 32

- Fichier de test: "sensor1.py" permet de récupérer les mesures du capteur de température & humidité

		'Quick example for the i2c driver.'

		def run_sensor1():
		    '''Runs all of the methods from the i2c driver. Imports are included in the
		    method for re-importing any updates when testing.
		    '''
		    import si7021
		    import machine
		    from time import sleep_ms
		    i2c = machine.I2C(machine.Pin(0), machine.Pin(2))

		    temp_sensor = si7021.Si7021(i2c)
		    print('Serial:              {value}'.format(value=temp_sensor.serial))
		    print('Identifier:          {value}'.format(value=temp_sensor.identifier))
		    print('Temperature:         {value}'.format(value=temp_sensor.temperature))
		    print('Relative Humidity:   {value}'.format(value=temp_sensor.relative_humidity))
		    print('Fahrenheit:          {value}'.format(value=si7021.convert_celcius_to_fahrenheit(temp_sensor.temperature)))
    
		    #sleep_ms(5000)
		    #print('Temperature:         {value}'.format(value=temp_sensor.temperature))


- Library uRequests: fichier LIB "urequests.py"

		import usocket

		class Response:

		    def __init__(self, f):
		        self.raw = f
		        self.encoding = "utf-8"
		        self._cached = None

		    def close(self):
		        if self.raw:
		            self.raw.close()
		            self.raw = None
		        self._cached = None

		    @property
		    def content(self):
		        if self._cached is None:
		            try:
		                self._cached = self.raw.read()
		            finally:
		                self.raw.close()
		                self.raw = None
		        return self._cached

		    @property
		    def text(self):
		        return str(self.content, self.encoding)

		    def json(self):
		        import ujson
		        return ujson.loads(self.content)


		def request(method, url, data=None, json=None, headers={}, stream=None):
		    try:
		        proto, dummy, host, path = url.split("/", 3)
		    except ValueError:
		        proto, dummy, host = url.split("/", 2)
		        path = ""
		    if proto == "http:":
		        port = 80
		    elif proto == "https:":
		        import ussl
		        port = 443
		    else:
		        raise ValueError("Unsupported protocol: " + proto)

		    if ":" in host:
		        host, port = host.split(":", 1)
		        port = int(port)

		    ai = usocket.getaddrinfo(host, port, 0, usocket.SOCK_STREAM)
		    ai = ai[0]

		    s = usocket.socket(ai[0], ai[1], ai[2])
		    try:
		        s.connect(ai[-1])
		        if proto == "https:":
		            s = ussl.wrap_socket(s, server_hostname=host)
		        s.write(b"%s /%s HTTP/1.0\r\n" % (method, path))
		        if not "Host" in headers:
		            s.write(b"Host: %s\r\n" % host)
		        # Iterate over keys to avoid tuple alloc
		        for k in headers:
		            s.write(k)
		            s.write(b": ")
		            s.write(headers[k])
		            s.write(b"\r\n")
		        if json is not None:
		            assert data is None
		            import ujson
		            data = ujson.dumps(json)
		            s.write(b"Content-Type: application/json\r\n")
		        if data:
		            s.write(b"Content-Length: %d\r\n" % len(data))
		        s.write(b"\r\n")
		        if data:
		            s.write(data)

		        l = s.readline()
		        #print(l)
		        l = l.split(None, 2)
		        status = int(l[1])
		        reason = ""
		        if len(l) > 2:
		            reason = l[2].rstrip()
		        while True:
		            l = s.readline()
		            if not l or l == b"\r\n":
		                break
		            #print(l)
		            if l.startswith(b"Transfer-Encoding:"):
		                if b"chunked" in l:
		                    raise ValueError("Unsupported " + l)
		            elif l.startswith(b"Location:") and not 200 <= status <= 299:
		                raise NotImplementedError("Redirects not yet supported")
		    except OSError:
		        s.close()
		        raise

		    resp = Response(s)
		    resp.status_code = status
		    resp.reason = reason
		    return resp


		def head(url, **kw):
		    return request("HEAD", url, **kw)

		def get(url, **kw):
		    return request("GET", url, **kw)

		def post(url, **kw):
		    return request("POST", url, **kw)

		def put(url, **kw):
		    return request("PUT", url, **kw)

		def patch(url, **kw):
		    return request("PATCH", url, **kw)

		def delete(url, **kw):
		    return request("DELETE", url, **kw)

- Fichier de test: "post_api.py" permet la connexion + permet d'envoyer les relevées du capteur via une requêtes à l'api 

		from machine import Pin
		from time import sleep
		import urequests as requests
		import json
		import si7021
		import machine

		try:
		  import usocket as socket
		except:
		  import socket

		from machine import Pin
		import network

		import esp
		esp.osdebug(None)

		import gc
		gc.collect()

		ssid = 'Livebox-5476'
		password = 'Clemlgy76'

		station = network.WLAN(network.STA_IF)

		station.active(True)
		station.connect(ssid, password)

		while station.isconnected() == False:
		  pass

		print('Connection successful')
		print(station.ifconfig())

		'Quick example for the i2c driver.'

		def run_sensor1():
		    i2c = machine.I2C(machine.Pin(0), machine.Pin(2))

		    temp_sensor = si7021.Si7021(i2c)
		    print('Serial:              {value}'.format(value=temp_sensor.serial))
		    print('Identifier:          {value}'.format(value=temp_sensor.identifier))
		    print('Temperature:         {value}'.format(value=temp_sensor.temperature))
		    print('Relative Humidity:   {value}'.format(value=temp_sensor.relative_humidity))
		    print('Fahrenheit:          {value}'.format(value=si7021.convert_celcius_to_fahrenheit(temp_sensor.temperature)))



		while True:
		  try:
		    i2c = machine.I2C(machine.Pin(0), machine.Pin(2))
		    temp_sensor = si7021.Si7021(i2c)
		    sensor = value=temp_sensor.identifier
		    temperature = temp_sensor.temperature
		    humidity = temp_sensor.relative_humidity
		    fahrenheit = si7021.convert_celcius_to_fahrenheit(temp_sensor.temperature)
		    data = {"temperature":temperature, "humidite":humidity, "farenheit":fahrenheit, "capture":sensor}
		    print(data)
		    json_data = json.dumps(data).encode('utf-8')
		    headers = {'Content-Type':'application/json'}
		    print(data)
		    print(station.isconnected())
		    r = requests.post("http://192.168.43.60:5000/api/v1/ajouter/", data=json_data, headers=headers)
		    #r = requests.post("http://127.0.0.1:5000/api/v1/donnees/", json = data)
		    #json_body= json.loads(r.text)
		    json_body= r
		    print(json_body)
		    print('Temperature: %3.1f' %temperature)
		    print('Humidity: %3.1f' %humidity)
		    del temperature, humidity, data, json_data
		    sleep(10)
		  except OSError as e:
		    print('Failed')
		    print(repr(e))


- Library LCD: fichier LIB "lcdi2c.py"

		# SPDX-FileCopyrightText: 2016 Scott Shawcroft for Adafruit Industries
		#
		# SPDX-License-Identifier: MIT

		"""
		`adafruit_bus_device.i2c_device` - I2C Bus Device
		====================================================
		"""

		__version__ = "0.0.0-auto.0"
		__repo__ = "https://github.com/adafruit/Adafruit_CircuitPython_BusDevice.git"


		class I2CDevice:
		    """
		    Represents a single I2C device and manages locking the bus and the device
		    address.
		    :param ~busio.I2C i2c: The I2C bus the device is on
		    :param int device_address: The 7 bit device address
		    :param bool probe: Probe for the device upon object creation, default is true
		    .. note:: This class is **NOT** built into CircuitPython. See
		      :ref:`here for install instructions <bus_device_installation>`.
		    Example:
		    .. code-block:: python
		        import busio
		        from board import *
		        from adafruit_bus_device.i2c_device import I2CDevice
		        with busio.I2C(SCL, SDA) as i2c:
		            device = I2CDevice(i2c, 0x70)
		            bytes_read = bytearray(4)
		            with device:
		                device.readinto(bytes_read)
		            # A second transaction
		            with device:
		                device.write(bytes_read)
		    """

		    def __init__(self, i2c, device_address, probe=True):

		        self.i2c = i2c
		        self.device_address = device_address

		        if probe:
		            self.__probe_for_device()

		    def readinto(self, buf, *, start=0, end=None):
		        """
		        Read into ``buf`` from the device. The number of bytes read will be the
		        length of ``buf``.
		        If ``start`` or ``end`` is provided, then the buffer will be sliced
		        as if ``buf[start:end]``. This will not cause an allocation like
		        ``buf[start:end]`` will so it saves memory.
		        :param bytearray buffer: buffer to write into
		        :param int start: Index to start writing at
		        :param int end: Index to write up to but not include; if None, use ``len(buf)``
		        """
		        if end is None:
		            end = len(buf)
		        self.i2c.readfrom_into(self.device_address, buf, start=start, end=end)

		    def write(self, buf, *, start=0, end=None):
		        """
		        Write the bytes from ``buffer`` to the device, then transmit a stop
		        bit.
		        If ``start`` or ``end`` is provided, then the buffer will be sliced
		        as if ``buffer[start:end]``. This will not cause an allocation like
		        ``buffer[start:end]`` will so it saves memory.
		        :param bytearray buffer: buffer containing the bytes to write
		        :param int start: Index to start writing from
		        :param int end: Index to read up to but not include; if None, use ``len(buf)``
		        """
		        if end is None:
		            end = len(buf)
		        self.i2c.writeto(self.device_address, buf, start=start, end=end)

		    # pylint: disable-msg=too-many-arguments
		    def write_then_readinto(
		        self,
		        out_buffer,
		        in_buffer,
		        *,
		        out_start=0,
		        out_end=None,
		        in_start=0,
		        in_end=None
		    ):
		        """
		        Write the bytes from ``out_buffer`` to the device, then immediately
		        reads into ``in_buffer`` from the device. The number of bytes read
		        will be the length of ``in_buffer``.
		        If ``out_start`` or ``out_end`` is provided, then the output buffer
		        will be sliced as if ``out_buffer[out_start:out_end]``. This will
		        not cause an allocation like ``buffer[out_start:out_end]`` will so
		        it saves memory.
		        If ``in_start`` or ``in_end`` is provided, then the input buffer
		        will be sliced as if ``in_buffer[in_start:in_end]``. This will not
		        cause an allocation like ``in_buffer[in_start:in_end]`` will so
		        it saves memory.
		        :param bytearray out_buffer: buffer containing the bytes to write
		        :param bytearray in_buffer: buffer containing the bytes to read into
		        :param int out_start: Index to start writing from
		        :param int out_end: Index to read up to but not include; if None, use ``len(out_buffer)``
		        :param int in_start: Index to start writing at
		        :param int in_end: Index to write up to but not include; if None, use ``len(in_buffer)``
		        """
		        if out_end is None:
		            out_end = len(out_buffer)
		        if in_end is None:
		            in_end = len(in_buffer)

		        self.i2c.writeto_then_readfrom(
		            self.device_address,
		            out_buffer,
		            in_buffer,
		            out_start=out_start,
		            out_end=out_end,
		            in_start=in_start,
		            in_end=in_end,
		        )

		    # pylint: enable-msg=too-many-arguments

		    def __enter__(self):
		        while not self.i2c.try_lock():
		            pass
		        return self

		    def __exit__(self, exc_type, exc_val, exc_tb):
		        self.i2c.unlock()
		        return False

		    def __probe_for_device(self):
		        """
		        Try to read a byte from an address,
		        if you get an OSError it means the device is not there
		        or that the device does not support these means of probing
		        """
		        while not self.i2c.try_lock():
		            pass
		        try:
		            self.i2c.writeto(self.device_address, b"")
		        except OSError:
		            # some OS's dont like writing an empty bytesting...
		            # Retry by reading a byte
		            try:
		                result = bytearray(1)
		                self.i2c.readfrom_into(self.device_address, result)
		            except OSError:
		                # pylint: disable=raise-missing-from
		                raise ValueError("No I2C device at address: 0x%x" % self.device_address)
		                # pylint: enable=raise-missing-from
		        finally:
		            self.i2c.unlock()


- Fichier de test: "lcd.py" permet d'afficher sur l'écran LCD le message "Hello, from MicroPython !"
- Attention: Ne pas oublier de faire le réglage  du contraste de l'écran qui se trouve à l'arrière !!

		import machine
		from machine import I2C, Pin
		from lcdi2c import LCDI2C
		from time import sleep

		# Pyboard - SDA=Y10, SCL=Y9
		#i2c = I2C(2)
		# ESP8266 sous MicroPython
		i2c = I2C(scl=Pin(0), sda=Pin(2))

		# Initialise l'ecran LCD
		lcd = LCDI2C( i2c, cols=16, rows=2 )
		lcd.backlight()

		lcd.set_cursor( (0, 0) ) # Tuple with Col=4, Row=1, zero based indexes

		# display a message (no automatic linefeed)
		lcd.print("Hello, from MicroPython !")
		sleep(1)
		# horizontal scrolling
		for i in range( 10 ):
			lcd.scroll_display()
			sleep( 0.500 )
	
		lcd.clear()

- Mettre dans le Fichier « boot.py » le programme FINALE, pour le faire booter directement au démarrage de l’ESP8266 :

		from time import sleep
		import urequests as requests
		import json
		import machine
		import time
		import network
		import esp
		import si7021
		from machine import I2C, Pin
		from lcdi2c import LCDI2C


		try:
		  import usocket as socket
		except:
		  import socket

		esp.osdebug(None)

		import gc
		gc.collect()

		ssid = 'reseau_nom’
		password = 'reseau_mdp'

		station = network.WLAN(network.STA_IF)

		station.active(True)
		station.connect(ssid, password)

		while station.isconnected() == False:
		  pass

		# print('Connection successful')
		# print(station.ifconfig())
		sec = time.localtime()[5]
		mi = time.localtime()[4]
		h = time.localtime()[3]
		d = time.localtime()[2]
		m = time.localtime()[1]
		y = time.localtime()[0]



		print(str(d) + "/"+ str(m) +"/"+ str(y) +" - "+ str(h) +":" + str(mi) +":"+ str(sec))
		'Quick example for the i2c driver.'

		def run_sensor1():
		    i2c = machine.I2C(machine.Pin(0), machine.Pin(2))

		    temp_sensor = si7021.Si7021(i2c)
		    print('Serial:              {value}'.format(value=temp_sensor.serial))
		    print('Identifier:          {value}'.format(value=temp_sensor.identifier))
		    print('Temperature:         {value}'.format(value=temp_sensor.temperature))
		    print('Relative Humidity:   {value}'.format(value=temp_sensor.relative_humidity))
		    print('Fahrenheit:          {value}'.format(value=si7021.convert_celcius_to_fahrenheit(temp_sensor.temperature)))


		def get_measures(mode="all", measures=5, interval=1):

		        i2c = I2C(scl=Pin(0), sda=Pin(2))
		        s = si7021.Si7021(i2c)
		        temp_data = []
		        humi_data = []
		        fahr_data = []
		        #print("On va prendre "+str(measures)+" mesures a "+str(interval)+" secondes d'intervalle")
		        while measures > 0:
		            #print("mesure restante "+str(measures)+"\nTemperature:"+str(s.temperature())+"\nHumidite:"+str(s.humidity()))
		            temp_data.append(s.temperature)
		            humi_data.append(s.relative_humidity)
		            fahr_data.append(si7021.convert_celcius_to_fahrenheit(s.temperature))
		            measures -= 1
		            time.sleep(interval)
		        print({"all temp":temp_data})
		        print({"all hum":humi_data})
		        print({"all fahr": fahr_data})
		        temperature = round(sum(temp_data)/len(temp_data),1)
		        humidite = round(sum(humi_data)/len(humi_data),1)
		        fahrenheit = round(sum(fahr_data)/len(fahr_data),1)
		        if mode == "all":
		            data = {"temperature": temperature, "humidite": humidite, "farenheit":fahrenheit, "capture":s.identifier}
		        elif mode == "temperature":
		            data = {"temperature": temperature}
		        elif mode == "humidite":
		            data = {"humidite": humidite}
		        elif mode == "farenheit":
		            data = {"farenheit": fahrenheit}
		        else:
		            data = {"temperature": temperature, "humidite": humidite, "farenheit": fahrenheit, "capture":s.identifier}
		        return data

		i2c = I2C(scl=Pin(0), sda=Pin(2))
		lcd = LCDI2C( i2c, cols=16, rows=2 )
		lcd.backlight()
		lcd.set_cursor( (0, 0) ) # Tuple with Col=4, Row=1, zero based indexes
		# display a message (no automatic linefeed)
		lcd.print("Bienvenue sur notre station meteo !")
		# horizontal scrolling
		for i in range( 20 ):
		    lcd.scroll_display()
		    sleep( 0.25 )
		lcd.clear()

		res = station.ifconfig()[0]
		lcd.backlight()
		lcd.set_cursor( (0, 0) ) 
		lcd.print("IP: ")
		lcd.set_cursor( (0, 1) )
		lcd.print(res)
		sleep( 0.5 )

		while True:
		    try:
		        data = get_measures(mode="all", measures=5, interval=1)
		        i2c = I2C(scl=Pin(0), sda=Pin(2))
		        s = si7021.Si7021(i2c)
		        #data = {"temperature":temperature, "humidite":humidity, "farenheit":fahrenheit, "capture":sensor}
		        print(data)
		        json_data = json.dumps(data).encode('utf-8')
		        headers = {'Content-Type':'application/json'}
		        print(station.isconnected())
		        r = requests.post("http://192.168.43.60:5000/api/v1/ajouter/", data=json_data, headers=headers)
		        json_body= json.loads(r.text)
		        print(json_body)
		        print('Temperature: %3.1f' %s.temperature)
		        print('Humidite: %3.1f' %s.relative_humidity)
        
		        lcd = LCDI2C( i2c, cols=16, rows=2 )
		        lcd.backlight()
		        lcd.set_cursor( (0, 0) )
		        lcd.print( "capteur: "+str(s.identifier))
		        lcd.set_cursor( (0, 1) )
		        lcd.print(str(d) + "/"+ str(m) +"/"+ str(y) +" - "+ str(h) +":" + str(mi) +":"+ str(sec))
		        sleep( 1 )
		        for i in range( 10 ):
		            lcd.scroll_display()
		            sleep( 0.5 )
		        lcd.clear()
		        lcd.set_cursor( (0, 0) )
		        lcd.print("temperature: "+ str(s.temperature)+"°C") 
		        lcd.set_cursor( (0, 1) )
		        lcd.print("fahrenheit: "+ str(si7021.convert_celcius_to_fahrenheit(s.temperature))+"°F")
		        for i in range( 20 ):
		            lcd.scroll_display()
		            sleep( 0.5 )
		        lcd.clear()
		        lcd.set_cursor( (0, 0) )
		        lcd.print("humidite: "+ str(s.relative_humidity)+"%")
		        sleep( 1 )
		        lcd.clear()
		        lcd.print(json_body)
		        sleep(1)
		        for i in range( 30 ):
		            lcd.scroll_display()
		            sleep( 0.5 )
		        lcd.clear()    

		        del  data, json_data
		        sleep(10)
		    except OSError as e:
		       print('Failed')
		       print(repr(e))
