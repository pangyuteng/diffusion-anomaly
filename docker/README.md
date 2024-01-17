
--

pip3 install visdom
visdom -port 8850

--

docker build -t pangyuteng/diffusion-anomaly .
docker push pangyuteng/diffusion-anomaly

docker run -it --network=host -u $(id -u):$(id -g) -v /mnt:/mnt -w $PWD pangyuteng/diffusion-anomaly bash


