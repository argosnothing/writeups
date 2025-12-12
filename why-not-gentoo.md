
> hi nix users
  why do you choose nix over gentoo
      i get why arch users and ubuntu users etc choose their distros but why nix over gentoo? 
      (not tryna start shit so don't let it come across like that, just genuinely curious)
      @𝕯𝖗𝖆𝖌𝕽𝖚𝖓𝖊@Gentoo

I'll answer the question with another question. Why "over gentoo?"? What do you think gentoo provides that makes it attractive to the kind of person that would choose NixOS? Gentoo gives you more flexibility in the kind of packages you can put together, which does indeed let you dodge dependency issues between a set of packages that might have issues elsewhere (with work and crosschecking versions). This is the closest I can get to gentoo helping with an issue NixOS addresses, but even then it doesn't actually fix the issue of dependencies on a system.  

I think the answer to your question, not over gentoo, but over literally any non CAD style Operating system is in the [original thesis](https://edolstra.github.io/pubs/phd-thesis.pdf). If you're genuinely interested in why someone would run an entire system like this for your OS. It includes direct comparisons to popular package managers that portage is in the same category as. It also mentions portage.  

NixOS and Nix goes beyond even this though, it offers a truly unified linux operating system through allowing you to abstract over virtually any configuration file you want, and then tie the configuration to the package installation and it needing to be on the system itself. This is going to sound like mumbo jumbo to you so I'll give a concrete example: 

```nix

      (pkgs.writeShellScriptBin "record-region" ''
        FILE="$(mktemp --suffix=.mp4)"
        SRC="$(${pkgs.pulseaudio}/bin/pactl get-default-sink).monitor"
        ${pkgs-stable.wf-recorder}/bin/wf-recorder \
          -g "$(${pkgs-stable.slurp}/bin/slurp)" \
          --audio="$SRC" \
          --audio-backend=pipewire \
          -C aac \
          -f "$FILE" -y
        printf 'file://%s\n' "$FILE" | wl-copy --type text/uri-list
      '')
```

In this one bit of nix code we're actually doing a few things, we are

1. Creating a shell script that will be available in our $PATH
2. Creating a implementation for what the script does by interpolating bash
3. On-the-fly including packages that might not even be on the system yet by resolving them to their store paths. ( based on the current nixpkgs version we have locked in our flake )

This is binding the actual configuration for some program to those dependencies being installed. If i handed the snippet of code to someone else, as long as they had nixpkgs in their scope, when they rebuild with this it will just work. Configuration gets locked to packages, no more "you need xyz dependency". We ensure the dependencies exist in our dotfiles. It handles all of that. 

Another example, go and install nix on your machine, the package manager, then run 
`nix run github:argosnothing/nixos-config#nvf`  
You will find my exact current configuration of nvim, locked to the dependencies, just like I'm currently using it, lsp included. Nix packages and outputs to our configurations are truly portable. It will just work on your machine, with or without those packages installed through your system... How the fuck?? Hopefully you've looked through the thesis and you have an idea on how we're able to accomplish this level of portability with 0 reliance on containerization. Our systems and packages are perfectly crystalline. "Nix is a prison" is the biggest lie I've seen. We, just as a sideeffect of how nix works, have a universal package manager, for anyone to use when they install our nix package manager, and it doesn't even touch your own packages in the process. 

I'm a developer, I like playing around with a lot of projects, however niri's versions of its package set *WILL* be different than other rust projects i'm working on. Without reliance on containerization like docker, with it's own headache things to learn, I can instead leverage my nix knowledge and simply create a flake in the project root that describes the versions I need to have when entering. Any system-wide rust relating tooling I have ( i don't have any ) will be shadowed by the ones declared in that local project. I can jump around projects of different versions and use them with the same degree of ergonomics that they'd have if they were system installed. 

In the same vein, a nix-config takes on the form of a software project, as opposed to the traditional dotfiles of random config templates, this means I can logically partition and organize everything in a way that makes sense *to me*. I don't need to create a ~/.config/xyz structure so it maps cleanly onto those directories, or use tools like stow and ensure they stick to those structures. I make the structure of my config to what makes sense to me. And again, all of my config is coupled with packages and their versions, perfectly crystalline. 

I'm also a tinkerer, and i love to play around with different software, specifically window managers. This doesn't mean I want them all installed at once, or even their configs in my user space, without needing stow **or** contorting myself to xdg_config conventions I can only have window manager config ( or any configured package ) in my home directory that I am currently using or testing, all while **actually** conforming to those conventions in the instantiated config. 

Following image is a example of my nix file structure. Note the amount of window managers and software configurations; I don't have anything but niri actually installed ( taking up room on my system ), but with a simple module import I can bring both a new window manager, and my own fully configured "dotfile" for it.   
<img width="328" height="1213" alt="image" src="https://github.com/user-attachments/assets/3c29190c-2f9b-4506-904e-d6072820ec99" />


With NixOS I feel like I can make it any system I want at any moment, because I have architected it in such a way that it makes sense to me. It lets me create my own abstractions that I can then apply to various configs. Case in point: 

```nix

    my = {
      cursor.speed = -0.35;
      theme.polarity = "dark";
      fonts.size = 11;
      is-vm = false;
      monitors = [
        {
          name = "DP-1";
          is-primary = true;
          dimensions = {
            width = 3840;
            height = 2160;
          };
          position = {
            x = 0;
            y = 0;
          };
          scale = 1.2;
          refresh = 143.851;
        }
        {
          name = "DP-2";
          dimensions = {
            width = 1920;
            height = 1080;
          };
          position = {
            x = 3202;
            y = 402;
          };
          scale = 1.0;
          refresh = 60.0;
        }
      ];
    };

```

Here I am populating a set of options that I myself created, for a particular host that has two displays. Each window manager config I have created then maps this information into their own configuration in whatever format they need. I don't need to concern myself with how they do it, all I need to do is tell my config that this host is the one with these details. I could even make this more ergonomic, and have nix actually compute positions based on the size and scale, as I've seen other nixers do. 


All of this is fairly technical, and it all requires a lot of architecting and work. I **Love** nix, I love writing it, I love being able to have 4 machines that I can keep up to date at the same time. 


The list will go on for at least 5 more reasons, there is increased integrity with built in closure rollbacks, there is the obvious boon of atomic updates, there is the flexibility in that I could use flags and not rely on the nix cache at all if I wanted, there is the easy interop between other peoples configs and their outputs, oh yeah transient nix shells are god; I can just test software that I don't even have installed like magic. I can take an aspect of the config through only the output and directly use it as I showed earlier. My config has already doubled as a way to instantly get my nvim on other machines when I just want my shit. 


Honestly, and because you entered the nix channel asking why *over gentoo*, of all things, I have to turn it back to you, why in the world should you care? Like do you have a small computer with not a lot of memory? Why obsess over compiler flags for different features, it makes your system more tailored to the use case ( after a ton of work ), but then you end up losing all that time just having to compile things anyway. It's like buying expensive solar panels because it will save you money in the long term, but the calculus rarely pans out that way; all the time spent not being able to use your computer you could have put into learning some skill that makes you better. NixOS also suffers from a massive amount of investment to actually really make it your own, but then the end product is a perfectly preserved config that you will always have, just as you left it. 
