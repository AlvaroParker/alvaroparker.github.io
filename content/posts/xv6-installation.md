+++
title="Como instalar xv6-riscv en Linux"
date="2025-01-06T17:09:15.000Z"

[taxonomies]
tags = ["Linux", "xv6", "riscv"]
+++

# Instalacion de dependencias xv6-riscv

Este gist es una guia rapida para instalar las dependencias necesarias para compilar y correr el kernel de xv6-riscv. Se describen los pasos para instalar las dependencias en Fedora, Debian, Ubuntu, Ubuntu WSL, Arch Linux y MacOS.

## Fedora

```bash
sudo dnf install binutils-riscv64-linux-gnu gcc-riscv64-linux-gnu make qemu-system-riscv git gcc
```

## Debian, Ubuntu y Ubuntu WSL

```bash
sudo apt install git make binutils-riscv64-linux-gnu gcc-riscv64-linux-gnu qemu-system-misc gcc
```

## Arch

```bash
sudo pacman -S qemu-system-riscv riscv64-linux-gnu-binutils riscv64-linux-gnu-gcc make git gcc

```

## MacOS

Para MacOS puedes instalar las dependencias con [brew](https://brew.sh/)

# Clonar repositorio y compilar

Clona el repositorio oficial de xv6 o el fork que tengas en tu propio repositorio

```bash
git clone https://github.com/mit-pdos/xv6-riscv.git
```

Entra al directorio y compila el kernel:

```bash
cd xv6-riscv
make qemu # Para correr el kernel en qemu
```

Una vez que el kernel este corriendo en qemu, puedes cerrar la ventana de qemu con `Ctrl + a` y luego `x`.
