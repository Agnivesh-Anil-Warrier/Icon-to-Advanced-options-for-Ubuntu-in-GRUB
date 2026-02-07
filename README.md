# Add an Icon to “Advanced options for Ubuntu” in GRUB

<img width="1500" height="551" alt="image" src="https://github.com/user-attachments/assets/b191635e-6bb9-4fa9-ace5-10a06febab92" />



Most GRUB themes can show icons for menu entries, but the
**“Advanced options for Ubuntu”** submenu has no class by default —
which means no icon appears even if your theme supports it.

This guide shows how to safely add one.

**Works on:** all modern versions of Ubuntu and Debian-based systems.

---

## Why this is needed

GRUB themes can assign icons using entry classes:

```
--class ubuntu  → ubuntu.png
--class windows → windows.png
```

But the **Advanced options** submenu is generated **without a class**,
so themes have nothing to map an icon to.

**Solution:** inject a class during menu generation.

---

## 1. Verify your theme is active

```bash
grep theme /boot/grub/grub.cfg
```

Expected output:
(something similar to)
```
set theme=/boot/grub/themes/mytheme/theme.txt
```

Confirm icons directory:
(something similar to)
```
/boot/grub/themes/mytheme/icons/
```

Icon must be:
(the naming is case-sensitive)
```
advanced.png
```

### Requirements

* lowercase
* PNG
* square (64×64 or 128×128)

---

## 2. Quick method (fastest)

### Edit generator

```bash
sudo nano /etc/grub.d/10_linux
```

Search for:

```
Advanced options
```

Find:

```bash
echo "submenu '$(gettext_printf "Advanced options for %s" "$OS")' | grub_quote) \$menuentry_id_option 'gnulinux-advanced-$boot_device_id' {"
```

Change to (Modify exactly like this and you may notice a color change):

```bash
echo "submenu '$(gettext_printf "Advanced options for %s" "$OS")' | grub_quote) \
--class advanced \
$menuentry_id_option 'gnulinux-advanced-$boot_device_id' {"
```

---

### Regenerate GRUB

```bash
sudo update-grub
```

Verify:

```bash
grep submenu /boot/grub/grub.cfg
```

You should see something like:

```
submenu 'Advanced options for Ubuntu' --class advanced ...
```

---

### Add icon

```
/boot/grub/themes/mytheme/icons/advanced.png
```

Reboot → icon appears.

---

## Icon mapping rules

Icons go in:

```
/boot/grub/themes/<theme>/icons/
```

---

## Debug checklist

Check class exists:

```bash
grep advanced /boot/grub/grub.cfg
```

Check theme path:

```bash
grep theme /boot/grub/grub.cfg
```

Check icon file:

```bash
ls /boot/grub/themes/mytheme/icons/
```

Permissions:

```bash
sudo chmod 644 advanced.png
```

---

## How GRUB builds menus (mental model)

```
/etc/grub.d scripts
        ↓
update-grub
        ↓
/boot/grub/grub.cfg
        ↓
GRUB loads theme
        ↓
class → icon mapping
```

---

## What this enables

Once we know class injection, we can add icons for:

* recovery entries
* memtest
* specific kernels
* custom ISO boots
* other submenus
