{
  description = "Minimal LXQt NixOS Configuration";

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";
  };

  outputs = { self, nixpkgs, ... }@inputs: {
    nixosConfigurations = {
      "lxqt-vm" = nixpkgs.lib.nixosSystem {  # Name your configuration
        system = "x86_64-linux";
        modules = [
          ({ config, pkgs, ... }: {
            # Base system configuration
            networking.hostName = "lxqt-nixos";
            networking.networkmanager.enable = true;
            time.timeZone = "Europe/Berlin";

            # Enable LXQt with minimal components
            services.xserver = {
              enable = true;
              desktopManager.lxqt.enable = true;
              displayManager.sddm.enable = true;
              libinput.enable = true;  # Touchpad/touchscreen support
            };

            # Essential utilities
            environment.systemPackages = with pkgs; [
              # Core LXQt components
              lxqt.lxqt-panel
              lxqt.pcmanfm-qt
              lxqt.lximage-qt
              
              # Basic applications
              firefox
              kitty  # Terminal
              neovim
            ];

            # Hardware support
            sound.enable = true;
            hardware.pulseaudio.enable = false;
            services.pipewire = {
              enable = true;
              alsa.enable = true;
              pulse.enable = true;
            };

            # Optional but recommended for desktop use
            services.udisks2.enable = true;  # Disk mounting
            services.printing.enable = true;  # CUPS
            programs.dconf.enable = true;     # GTK settings

            # Nix configuration
            nix.settings.experimental-features = [ "nix-command" "flakes" ];
            system.stateVersion = "23.11";
          })
        ];
      };
    };
  };
}
