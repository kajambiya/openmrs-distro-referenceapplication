#dev_3_ohri job, the tags property for the docker/build-push-action step is updated to include ${{ github.run_number }}, which is a unique identifier for each run. This will tag the dev image with an incremental count.

#working_3_ohri job, the tags property for the docker/build-push-action step is updated to include ${{ github.run_number }}, which will tag the image with the runner timestamp. Also, the if property is updated to check for the working_3_ohri branch, and the `needs

name: O3 OHRI Distro Docker CI/CD

on:
  push:
    tags:
      - "*"
    branches:
      - "*"

  pull_request:
    branches:
      - "*"
    types: [opened, synchronize]
  release:
    types:
      - created

jobs:
  working_3_ohri:
    runs-on: ${{ matrix.os }}

    strategy:
      matrix:
        os: [ubuntu-last, macos-latest]

    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/working_3_ohri' }}

    steps:
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v2
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/openmrs3-backend:working_${{ github.run_number }}
      - name: Save image digest
        run: echo "::set-output name=image::${{ steps.build-and-push.outputs.digest }}"

  dev_3_ohri:
    runs-on: ${{ matrix.os }}

    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]

    needs: working_3_ohri

    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/dev_3_ohri' }}

    steps:
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v2
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Build and push
        id: build-an-push
        uses: docker/build-push-action@v4
        with:
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/openmrs3-backend:dev-${{ github.run_number}}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      - name: Save image digest
        run: echo "::set-output name=image::${{ steps.build-and-push.output.digest }}"
