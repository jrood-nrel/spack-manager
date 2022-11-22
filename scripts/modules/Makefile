export SPACK_USER_CONFIG_PATH=$(CURDIR)/.spack
export SPACK_USER_CACHE_PATH=$(CURDIR)/.spack
unexport SPACK_DISABLE_LOCAL_CONFIG

.SUFFIXES:

.PHONY: all clean

all: software/generated/env

clean:
	rm -rf base/spack.mk base/spack.lock base/compilers.yaml base/modules.yaml base/config.yaml base/packages.yaml base/generated base/.spack-env
	rm -rf compilers/spack.mk compilers/spack.lock compilers/compilers.yaml compilers/modules.yaml compilers/config.yaml compilers/packages.yaml compilers/generated compilers/.spack-env
	rm -rf software/spack.mk software/spack.lock software/compilers.yaml software/modules.yaml software/config.yaml software/packages.yaml software/generated software/.spack-env

# Base compiler installation
base/compilers.yaml:
				SPACK_USER_CONFIG_PATH=base spack compiler find --scope=user

base/modules.yaml:
				SPACK_USER_CONFIG_PATH=base spack config --scope=user add modules:default:tcl:projections:all:"base/{compiler.name}-{compiler.version}/{name}/{version}/{hash:4}"

base/config.yaml:
				SPACK_USER_CONFIG_PATH=base spack config --scope=user add config:install_tree:root:$(CURDIR)/base_store

base/packages.yaml:
				SPACK_USER_CONFIG_PATH=base spack external find --scope=user perl m4 autoconf automake libtool gawk

base/spack.lock: base/spack.yaml base/config.yaml base/packages.yaml base/compilers.yaml base/modules.yaml
				spack -e base concretize --force

base/spack.mk: base/spack.lock
				spack -e base env depfile --make-target-prefix base/generated -o $@

# Main compiler installation
compilers/compilers.yaml: base/generated/env
				SPACK_USER_CONFIG_PATH=compilers spack compiler find --scope=user $$(spack -e base find --format '{prefix}' gcc)

compilers/modules.yaml:
				SPACK_USER_CONFIG_PATH=compilers spack config --scope=user add modules:default:tcl:projections:all:"compilers/{compiler.name}-{compiler.version}/{name}/{version}/{hash:4}"

compilers/config.yaml: 
				SPACK_USER_CONFIG_PATH=compilers spack config --scope=user add config:install_tree:root:$(CURDIR)/compilers_store

compilers/packages.yaml:
				SPACK_USER_CONFIG_PATH=compilers spack external find --scope=user perl m4 autoconf automake libtool gawk ncurses openssl

compilers/spack.lock: compilers/spack.yaml compilers/config.yaml compilers/packages.yaml compilers/compilers.yaml compilers/modules.yaml
				spack -e compilers concretize --force

compilers/spack.mk: compilers/spack.lock
				spack -e compilers env depfile --make-target-prefix compilers/generated -o $@

# Software installation
software/compilers.yaml: compilers/generated/env
				SPACK_USER_CONFIG_PATH=software spack compiler find --scope=user $$(spack -e compilers find --format '{prefix}' gcc llvm)

software/modules.yaml:
				SPACK_USER_CONFIG_PATH=software spack config --scope=user add modules:default:tcl:projections:all:"software/{compiler.name}-{compiler.version}/{name}/{version}/{hash:4}"

software/config.yaml:
				SPACK_USER_CONFIG_PATH=software spack config --scope=user add config:install_tree:root:$(CURDIR)/software_store

software/packages.yaml:
				SPACK_USER_CONFIG_PATH=software spack external find --scope=user perl m4 autoconf automake libtool gawk ncurses openssl openssh libxcrypt

software/spack.lock: software/spack.yaml software/config.yaml software/packages.yaml software/compilers.yaml software/modules.yaml

				spack python mixed_compiler_creator.py software/compilers.yaml
				spack -e software concretize --force

software/spack.mk: software/spack.lock
				spack -e software env depfile --make-target-prefix software/generated -o $@

ifeq (,$(filter clean,$(MAKECMDGOALS)))

include base/spack.mk

ifneq (,$(wildcard base/spack.mk))
include compilers/spack.mk
endif

ifneq (,$(wildcard compilers/spack.mk))
include software/spack.mk
endif

endif
