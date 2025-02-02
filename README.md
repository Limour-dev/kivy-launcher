# Kivy Launcher

(work in progress, not yet published on Google Play)

This is a reboot of the previously pygame/kivy launcher, implemented in Java in Python for android. It was barely maintainable, and with the rewrite of the new Python for android, it was lost.

This version aimed to provide a replacement for the launcher, but works also on desktop, on Python 2 or 3.

Anybody can clone the repo, add the dependencies we would not provide by default, and recompile it.

![kivy-launcher](https://user-images.githubusercontent.com/37904/37256979-0611d5be-2563-11e8-98a6-485e656b0f4b.png)

## How it works

Follow the guide the same as before:

https://kivy.org/docs/guide/packaging-android.html#packaging-your-application-for-the-kivy-launcher

Then just start the launcher, you should see your application listed, then press play.

## `android.txt` specification

- `title`: Title of the application
- `author`: Author of the application
- `orientation`: Default orientation, one of "landscape", "portrait", "sensor"

## Works

- Provide a simple UI to discover and start another app
- Start another main.py as a `__name__ == '__main__'`
- Reduce to the minimum the overhead of the launcher to launch another app
- Support landscape / portrait / sensor

## Test
```cmd
conda create -n kivy conda-forge::kivy=2.3.0 conda-forge::python=3.8
```
```cmd
java -jar AXMLPrinter2.jar AndroidManifest.xml > manifest.txt
chcp 65001
adb logcat | findstr org.kivy
adb shell
getprop ro.build.version.sdk
run-as org.kivy.launcher
cd files/app/.kivy/
cat logs/kivy_25-02-01_1.txt
```
```cmd
adb shell
run-as org.kivy.launcher
cd ~/files/app
mkdir -p kivy/debugger
cp -r /storage/emulated/0/kivy/debugger kivy
```
```python
from aiohttp import web
import aiofiles
from os.path import join
from threading import Thread
try:
    from android.storage import primary_external_storage_path
    root = join(primary_external_storage_path(), 'Download')
except Exception as e:
    root = '.'

def get_local_ips():
    try:
        import socket
        hostname = socket.gethostname()
        print(hostname)
        _, _, addr_list = socket.gethostbyname_ex(hostname)
        print(addr_list)
    except Exception as e:
        print(e)

async def handle(req):
    if req.method == 'GET':
        return web.Response(text='''
            <form method="post" enctype="multipart/form-data">
            <input type="file" name="file">
            <input type="submit">
            </form>
        ''', content_type='text/html')
    
    reader = await req.multipart()
    field = await reader.next()
    
    if field.name != 'file':
        return web.Response(text="No file uploaded", status=400)
    
    filename = join(root, field.filename)
    async with aiofiles.open(filename, 'wb') as f:
        while True:
            chunk = await field.read_chunk()
            if not chunk:
                break
            await f.write(chunk)
    
    return web.Response(text=f"{filename} uploaded!")

app = web.Application(client_max_size=20*1024**2)
app.add_routes([web.route('*', '/', handle)])

def main():
    get_local_ips()
    web.run_app(app, host='0.0.0.0', port=8888) 

Thread(target=main, daemon=True).start()
```
## Ideas

- Act as a server to just launch any Kivy-based app from desktop to mobile
- Ability to configure multiple paths to look for applications
- Different ordering: by name, last updated, size
- Add tiny icon to show what application orientation is
- Allow to change multiple configuration token / environemnt (like different density/dpi to simulate other screens)
- Support for application without "android.txt"
