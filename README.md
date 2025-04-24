{
  description = "Minimal LXQt NixOS Configuration";
 
  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";
  };
 
  outputs = { self, nixpkgs, ... }@inputs: {
    nixosConfigurations = {
      "g" = nixpkgs.lib.nixosSystem {
        system = "x86_64-linux";
        modules = [
          # Import generated hardware config (if exists)
          ./hardware-configuration.nix
          <nixos-hardware/microsoft/surface/surface-pro-intel>

           
          ({ config, pkgs, ... }: {
            # Base system configuration
            networking.hostName = "g";
            networking.networkmanager.enable = true;
            time.timeZone = "Europe/Kaliningrad";
 
#           # Essential boot configuration
#           boot.loader.grub = {
#             enable = true;
#             #device = "/dev/nvme0n1"; # Change to your disk device
#             useOSProber = true;
#           };

           
            boot.kernelPatches = [{
              name = "disable-rust";
              patch = null;
              extraConfig = ''
                RUST n
              '';
            }];

 
            # Bootloader.
            boot.loader.systemd-boot.enable = true;
            boot.loader.efi.canTouchEfiVariables = true;


            # Cinnamon
            services.xserver = {
              enable = true;
              desktopManager.cinnamon.enable = true;
              displayManager = {
                lightdm.enable = true;
                autoLogin.user = "g";
              };
              libinput.enable = true; # touch support?
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
            #  cinnamon.cinnamon
              nemo
              xed
              firefox
              kitty
              neovim
              git
              wget
            ];
 
            # Define a user account. Don't forget to set a password with ‘passwd’.
            users.users.g = { isNormalUser = true; description = "g"; initialPassword = "j"; extraGroups = [ "networkmanager" "wheel" ]; packages = with pkgs; [];
            };
 
 
 
            # Nix configuration
            nix.settings.experimental-features = [ "nix-command" "flakes" ];
            system.stateVersion = "24.11";
          })
        ];
      };
    };
  };
}
 
