## Fixing KeePassXC on Wayland

We just need to modify the desktop file so our application will be able to open KeePassXC with wayland support.

In order to do it, just find and modify that file as follows:

1. Find and modify the file:

```
cp /usr/share/applications/org.keepassxc.KeePassXC.desktop \
   ~/.local/share/applications/
```

2. Modify the exec within the desktop file:

```
nvim ~/.local/share/applications/org.keepassxc.KeePassXC.desktop
```

After those steps the application launchers should work as expected.
