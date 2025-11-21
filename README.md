# Chapter 0: Background knowledge
## Arch
Arch Linux is a lightweight, rolling-release Linux distribution built around simplicity and user control.
It provides only the bare minimum by default, and the user installs everything else manually.
Key features include the pacman package manager, the AUR for community packages, and a user-driven configuration model.

In your setup, Arch serves as the underlying system on which Hyprland, Waybar, Kitty, Swww, and Matugen all run.
## Hyprland
Hyprland is a dynamic tiling Wayland compositor.
Its roles include:

Managing windows, Handling animations and layout, Controlling workspaces, Handling inputs (keyboard, mouse), Loading its config from ~/.config/hypr/

Hyprland supports loading external color files, which is why Matugen outputs a generated colors.conf into Hyprland’s config directory.
When the file updates, Hyprland can be reloaded so the theme changes instantly.

## Waybar
Waybar is a customizable status bar for Wayland compositors.
It displays modules like time, system info, Hyprland active window, workspaces, etc.

Waybar uses two important files:

• config.jsonc — module layout
• style.css — styling and colors

Your Matugen setup generates a colors.css file that contains variables (such as background, accent, foreground).
Waybar’s style.css imports these color variables, so when Matugen regenerates colors.css, Waybar instantly updates after restart.

## Kitty
Kitty is a GPU-accelerated terminal emulator.
It allows external color configuration, usually via an included colors.conf file.

Matugen generates this colors.conf based on your wallpaper colors.
Kitty reloads it when Matugen triggers a signal in its post hook.
This lets your terminal theme change instantly with your wallpaper.

## Swww

Swww is a lightweight Wayland wallpaper program.
Matugen uses it in your setup to set wallpapers.

When you run something like:
matugen image <path>
Matugen passes arguments to swww to apply the wallpaper, optionally with transitions.
Once applied, Matugen extracts your wallpaper’s colors and generates theme files for everything else.

## GTK

GTK is the UI toolkit used by many graphical applications such as Thunar, File Roller, and Gedit.
Your setup includes a GTK template in Matugen, which outputs a CSS file containing themed colors.
This controls how GTK apps appear, making them match your system-wide colors.
## Matugen

Matugen is the central piece of your theming setup.
It reads colors from your wallpaper, then generates matching theme files for multiple programs.

In your configuration, Matugen handles:

• Reading the wallpaper
• Extracting colors
• Applying the wallpaper using swww
• Writing new theme files from templates
• Placing theme files in program-specific locations
• Optionally reloading programs automatically (post hooks)

Matugen templates define how each program receives these colors.
Matugen outputs:

• Hyprland colors
• Waybar colors
• Kitty colors
• GTK colors
• Rofi colors

## How Everything Interacts in Your Configuration

Below is the full pipeline of interaction, step by step.

You run Matugen with a wallpaper image.

Matugen sets the wallpaper using swww.

Matugen extracts colors from that wallpaper.

Matugen feeds those colors into templates for each program.

Matugen produces color theme files:
• ~/.config/hypr/colors.conf
• ~/.config/waybar/colors.css
• ~/.config/kitty/colors.conf
• ~/.config/gtk-3.0/gtk.css
• ~/.config/rofi/colors.rasi

Matugen writes these files to the correct locations.

Matugen runs post hooks if specified:
• Hyprland reload
• Waybar restart
• Kitty reload
• Thunar restart for GTK

When the programs reload:
• Hyprland windows adopt the new accent and background colors
• Waybar modules now use the updated CSS variables
• Kitty's terminal changes colors instantly
• GTK apps adopt new accent, background, and foreground colors
• Rofi menu matches the new theme

Everything on your system now looks consistent and matches the wallpaper.

# Important Paths and Their Roles in a Matugen Setup

## Matugen

### Config Matugen directory
    ~/.config/matugen/
### Matugen Tree
<pre>
matugen
├── config.toml
└── templates
    ├── colors.css
    ├── gtk-colors.css
    ├── hyprland-colors.conf
    ├── kitty-colors.conf
    └── rofi-colors.rasi
</pre>
    
### Matugen config directory
    ~/.config/matugen/config.toml 
Main Matugen configuration. Defines templates, output paths, wallpaper settings, and post hooks.
<pre>                 
[config.wallpaper]
# Whether to set the wallpaper or not
set = true

# The base command to run for applying the wallpaper, shouldn't have spaces in it.
command = "swww"

# The arguments that will be provided to the command.
# Keywords like {{ mode }} or anything that works inside of hooks doesn't work here.
# The last argument will be the image path.
arguments = ["img", "--transition-type", "center"]

[templates.kitty]
input_path = '~/.config/matugen/templates/kitty-colors.conf'
output_path = '~/.config/kitty/colors.conf'
post_hook = 'kill -SIGUSR1 $(pgrep kitty)'

[templates.hyprland]
input_path = '~/.config/matugen/templates/hyprland-colors.conf'
output_path = '~/.config/hypr/colors.conf'
#post_hook ='hyprctl reload'

[templates.rofi]
input_path = '~/.config/matugen/templates/rofi-colors.rasi'
output_path = '~/.config/rofi/colors/colors.rasi'

[templates.waybar]
input_path = '~/.config/matugen/templates/colors.css'
output_path = '~/.config/waybar/colors.css'
post_hook = 'pkill waybar; waybar & disown'

[templates.gtk]
input_path = '~/.config/matugen/templates/gtk-colors.css'
output_path = '~/.config/gtk-3.0/gtk.css'
post_hook = 'pkill Thunar; thunar & disown'
</pre>
### Breakdown of config.toml
#### [config.wallpaper]
This begins the wallpaper configuration section. Everything under this header tells Matugen how to handle setting your wallpaper.

#### set = true
This tells Matugen to actually set your wallpaper after generating colors. If this were false, Matugen would generate colors but not change the wallpaper.

#### command = "swww"
This tells Matugen which program to use for applying the wallpaper. In your case, you're using swww.

#### arguments = ["img", "--transition-type", "center"]
This lists the arguments Matugen will pass to swww when setting the wallpaper.
The resulting command (with the wallpaper file added at the end) will look like:
swww img --transition-type center <image>
"img" tells swww to set an image.
"--transition-type center" makes the transition animation originate from the center.

#### [templates.kitty]
This starts the section defining how Matugen should generate a color file for the Kitty terminal.

#### input_path = '~/.config/matugen/templates/kitty-colors.conf'
This is the template file Matugen uses to generate the Kitty color configuration. It contains variables that get replaced with the generated colors.

#### output_path = '~/.config/kitty/colors.conf'
This is the file Matugen will write the generated colors into. Kitty will load this file as your theme.

#### post_hook = 'kill -SIGUSR1 $(pgrep kitty)'
A command run after Matugen writes the new colors.
Sending SIGUSR1 to Kitty makes it reload its colors without needing to restart the terminal.

#### [templates.hyprland]
This section defines how Matugen generates Hyprland color config files.

#### input_path
This is the template file for Hyprland colors.

#### output_path
This is where the generated Hyprland color config is written.

#### #post_hook = 'hyprctl reload'
This is commented out, meaning unused. If you remove the #, Matugen would run "hyprctl reload" after generating colors, which forces Hyprland to reload its config.

#### [templates.rofi]
This defines how Matugen generates your Rofi theme colors.

#### input_path
Template file for Rofi colors.

#### output_path
The final Rofi color file that Rofi will use.
No post_hook here, because Rofi only reads its theme when it's launched, so nothing needs to be reloaded.

#### [templates.waybar]
This controls how Matugen generates the colors for Waybar.

#### input_path
Template file for Waybar colors (CSS).

#### output_path
The CSS file Waybar will load to apply your theme.

#### post_hook = 'pkill waybar; waybar & disown'
This restarts Waybar after updating the color file.
"pkill waybar" stops the old instance.
"waybar & disown" starts a new instance in the background.

#### [templates.gtk]
This controls GTK color generation. GTK colors affect applications like Thunar and anything using GTK3.

#### input_path
The template file for GTK colors.

#### output_path
The GTK CSS file Matugen overwrites with new colors.

#### post_hook = 'pkill Thunar; thunar & disown'
This closes Thunar and starts a new instance so the theme changes take effect immediately.

### Matugen Templates
    ~/.config/matugen/templates/ 
Contains all the template files for generating themes (Waybar, Kitty, hypr, etc).

## Hyprland config directory
### ~/.config/hypr/

colors.conf → Matugen-generated colors for Hyprland. Your template in Matugen produces this file.
#### hyprland.conf 
→ Main Hyprland configuration (general settings, compositor settings, input, etc.).
  
  hyprland.conf.save → Backup of your main config.
  
  hyprpaper.conf → Wallpaper management for Hyprland.
  
  monitors.conf → Monitor setup (resolution, refresh rate, arrangement).
  
  workspaces.conf → Workspace definitions, names, and layouts.
  
How it works together

Matugen reads the Hyprland template: 
### ~/.config/matugen/templates/hyprland-colors.conf.

Generates 
### ~/.config/hypr/colors.conf based on your theme and wallpaper.

In hyprland.conf, you reference colors.conf with a line like:
### source = ~/.config/hypr/colors.conf

This makes Hyprland use the Matugen theme automatically.

Post hook in config.toml can reload Hyprland or just the compositor to apply changes:
### post_hook = 'hyprctl reload'

## Waybar config directory
### ~/.config/waybar/

colors.css → the Matugen-generated theme file. Waybar reads this for colors.

config.jsonc → Waybar layout and module configuration (what’s left, center, right, workspaces, windows, clock, etc).

style.css → custom CSS for Waybar (borders, fonts, spacing, hover effects).

scripts/launch.sh → optional launch script for Waybar.

## Kitty config directory

### ~/.config/kitty/

colors.conf → the Matugen-generated color file for Kitty. Your templates in Matugen produce this.

### kitty.conf 
→ your main Kitty configuration. Usually contains general settings like font, scrollback, keybindings, and includes colors.conf with a line like:
include colors.conf
This makes Kitty use the colors generated by Matugen.

How it works together

Matugen reads the Kitty template: ~/.config/matugen/templates/kitty-colors.conf.

Generates ~/.config/kitty/colors.conf based on your current wallpaper and theme palette.

kitty.conf includes colors.conf, so Kitty automatically applies the theme.

Post hook in config.toml (like kill -SIGUSR1 $(pgrep kitty)) reloads Kitty so the new theme takes effect without restarting the terminal.
