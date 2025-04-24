{
  description = "Minimal LXQt NixOS Configuration";

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";
  };

  outputs = { self, nixpkgs, ... }@inputs: {
    nixosConfigurations = {
      "lxqt-vm" = nixpkgs.lib.nixosSystem {
        system = "x86_64-linux";
        modules = [
          # Import generated hardware config (if exists)
          ./hardware-configuration.nix

          ({ config, pkgs, ... }: {
            # Base system configuration
            networking.hostName = "lxqt-nixos";
            networking.networkmanager.enable = true;
            time.timeZone = "Europe/Berlin";

            # Essential boot configuration
            boot.loader.grub = {
              enable = true;
              device = "/dev/sda"; # Change to your disk device
              useOSProber = true;
            };

            # File systems (normally from hardware-configuration.nix)
            fileSystems."/" = {
              device = "/dev/disk/by-label/nixos"; # Set your actual device
              fsType = "ext4";
            };

            # LXQt configuration
            services.xserver = {
              enable = true;
              desktopManager.lxqt.enable = true;
              displayManager.sddm.enable = true;
              libinput.enable = true;
            };

            # Sound configuration (fixed)
            security.rtkit.enable = true;
            services.pipewire = {
              enable = true;
              alsa.enable = true;
              alsa.support32Bit = true;
              pulse.enable = true;
            };

            # Essential utilities
            environment.systemPackages = with pkgs; [
              lxqt.lxqt-panel
              lxqt.pcmanfm-qt
              lxqt.lximage-qt
              firefox
              kitty
              neovim
            ];

            # Nix configuration
            nix.settings.experimental-features = [ "nix-command" "flakes" ];
            system.stateVersion = "23.11";
          })
        ];
      };
    };
  };
}
