# net-core-systemd-service
A [systemd service](https://www.freedesktop.org/software/systemd/man/systemd.service.html) file template useful for .NET Core web servers behind an nginx reverse proxy.

## Requirements and Dependencies:

### 1. nginx as a reverse proxy server
This service will be dependent on [nginx](https://www.nginx.com/), because in my configuration(s), the .NET Core server is behind a reverse nginx proxy in order to take advantage of [pagespeed](https://developers.google.com/speed/pagespeed/module/) and not being restricted to just the .NET Core server being accessible.

### 2. Notify systemd upon successful initialization
In order for systemd to know that your server started correctly (instead of just hung), you will need to call [sd_notify](https://www.freedesktop.org/software/systemd/man/sd_notify.html) once your server is initialized. Here's how you can do that in C#:

```C#
using System.Runtime.InteropServices;

namespace MyCompany.SystemdStuff
{
    public static class Systemd
    {
        [DllImport("libsystemd.so.0")]
        public static extern int sd_notify(int unset_environment, string state);
    }
}
```
You call it like this:

```C#
using MyCompany.SystemdStuff;

    try {
        Systemd.sd_notify(0, "READY=1");
    } catch (Exception ex) {
        // do something because we're really boned.
    }
```
### 3. curl
In addition to the `sd_notify` call, systemd will use [curl](https://curl.haxx.se/) to kill two birds with one stone:
1. The .NET Core server gets warmed up by the GET request
2. If the command fails, systemd knows that something is wrong and your server initialized, but isn't working properly.

### 4. Kestrel
This template assumes that you're using Kestrel, and that your server listens on the [Unix domain socket](https://en.wikipedia.org/wiki/Unix_domain_socket) named `kestrel.sock`.

### 5. PID file
In order for systemd to know which process is your active server, you need to write the current [PID](https://en.wikipedia.org/wiki/Process_identifier) of the server process to the same file referenced in the `.service` file upon startup.

### 6. Permissions
Make sure that nginx, systemd (in the `.service` file), your .NET Core server, the location of the PID file, and the directory the server binary resides in are all executed and owned by the same user/group combination in the pursuit of security and uniformity. They'll also need execute privileges on the server binary and any other scripts you have as part of your implementation.

## Further reading
I would highly recommend reading the [systemd service documentation](https://www.freedesktop.org/software/systemd/man/systemd.service.html) to fully understand what each entry in the `.service` file means and what different values are possible.

## Errata
When I have some time, I will update this repository and README.md with further information and tips about how to fully implement this setup. I have also created some other tools and scripts that automate the process of building, uploading, and hot-swapping .NET Core server versions. You're gonna want those, but hopefully this saves you some time!
