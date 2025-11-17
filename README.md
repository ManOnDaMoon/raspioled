# raspioled
python library for 128 x 64 dot OLED display connected to Raspberry Pi via i2c

README translated from original repository (japanese)

<a href="https://github.com/h-nari/raspioled/wiki/images/181217a0.jpg">
<img src="https://github.com/h-nari/raspioled/wiki/images/181217a0.jpg" width="240"></a>

This is a Python library for a 128 x 64 dot OLED (controller is SSD1306) connected to a Raspberry Pi via one I2C.

I created this because I was dissatisfied with the slow drawing speed of the Adafruit_SSD1306.

## Features

- As with Adafruit_SSD1306, image data drawn with Pillow (image library) is transferred and displayed on the OLED.
- The process of converting Pillow.image.toBytes() to the OLED (SSD1306) format is implemented in C, so it is fast.
- The transfer of drawing data via i2c is done in a background thread, so the Python thread does not wait.
- Also implemented a function to wait for transfer completion

## Example

An example program is shown below.

    #!/usr/bin/python

    from RaspiOled import oled
    from PIL import Image,ImageDraw,ImageFont

    image = Image.new('1',oled.size)  # make 128x64 bitmap image
    draw  = ImageDraw.Draw(image)
    font = ImageFont.truetype(
        font='/usr/share/fonts/truetype/freefont/FreeMono.ttf',
        size=40)
    draw.text((0,0), "Hello", font=font, fill=255)
    oled.begin()
    oled.image(image)


## Installation

Using Python 3.
Requires `python3-dev` and `python3-setuptools`

    $ sudo python3 setup.py install
    $ sudo pip3 install Pillow

## Usage example

Please refer to the sample programs provided under examples.

When running the sample program, if there are any required libraries, please install them using pip etc. as appropriate.

## Set I2C speed to 400kHz

The default I2C transfer speed of the Raspberry Pi is 100kHz, which means that transferring image data takes about 100mS, which is not much different from the Adafruit_SSD1306.

Increasing the I2C speed to 400kHz will increase the transfer speed by about four times. The I2C speed seems to be determined by the kernel settings, so changes can be made in /boot/config.txt etc.

Add the following line to /boot/config.txt and reboot the Raspberry Pi.

    dtparam=i2c_baudrate=400000
    
## Properties

### **size**: 

Returns a tuple of the oled screen width and height, i.e. `(128,64)`

## Methods

### **begin(dev="/dev/i2c-1", i2c_addr=0x3c)**

Opens the i2c device file and starts the drawing sub-thread.


### **end()**

Terminates the sub-thread and closes the I2C device.


### **clear(update=1,sync=0,timeout=0.5,fill=0,area=(0,0,128,64))**

Clears the drawing buffer.

You can specify the fill color (0: black, 1: white) with fill.

You can specify the area to be cleared by specifying area. The method of specification is (x,y,w,h) or ((x,y),(w,h)).

If update=1, it will be transferred to oled immediately after clearing.

By setting sync=1, it will wait for the transfer to finish. By setting sync=0 as default, it will return immediately after clearing the buffer without waiting for the transfer to finish.

You can specify the maximum time to wait for the transfer to finish with timeout. The unit is seconds. If a timeout occurs, a TIMEOUT exception is raised in Python 3, and a rapioled.error exception is raised in Python 2.

### **image(image,dst_area=NULL,src_area=NULL,update=1,sync=0,timeout=0.5)**

PillPass in a Pillow image object and write it to the Oled drawing buffer.

image is a Pillow Image object. Its format must be bitmap, i.e. mode must be '1'.

You can specify the area to write to on the oled side with dst_area, and the area to read from on the image side with src_area. If not specified, the entire area will be targeted.

The region can be specified as (x,y), (x,y,w,h), or ((x,y),(w,h)). If w and h are not specified, the maximum possible width is assumed.

The update, sync, and timeout arguments are the same as for the clear method.

### **shift(amount=(-1,0),area=(0,0,128,64),fill=0,update=1,sync=0,timeout=0.5)**

Translation of the specified region on the OLED by the specified amount.

Amount specifies the amount of movement in x and y components. The default is (-1,0), which moves 1 dot to the left.

area specifies the area to shift. It must be in the format (x,y,w,h) or ((x,y)(w,h)). If not specified, the whole screen will be shifted.

fill specifies the color (0 or 1) to fill in the gaps created by the movement.

update, sync, and timeout are the same as the clear method.

### **vsync(timeout=0.5)**

Waits for the data transfer to oled after the clear and image_bytes methods.
If no data transfer has been performed, it returns immediately.
The timeout is the same as for the clear command.




    
