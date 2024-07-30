# Lidar Kullanarak Simülasyonda Lokalizasyon ve Haritalandırma (SLAM) ve Alınan Veriye Göre Otonom Robot Sürüşü

(Yazılım Kısmı)

# Proje Analizi
## PEAS Analizi:
Performans Kriterleri = Konumu Algılama, Haritalandırma, Patika Belirleme, Patika Takibi
Çevre = Raflar, Duvarlar (Çevre Analizi Kısmına Bakınız)
Hareket Mekanizmaları =Patika Takip Bilgisine Göre PID ile Yönetilen 3 Tekerli Holonomik Sürüş Sistemi
Sensörler = 2B Lidar, Encoderlar

## Çevre Analizi:
Bir Seans*	Partially	Stochastic	Sequental		Continuous
Birden Fazla Seans	Partially	Stochastic	Episodic		Continuous
* Seans, haritalandırmanın tamamlanmasından robotun kapanması arasında geçen süreyi belli eder
  
# Gerekli Programların Kurulması
## İşletim sistemi kurulumu
İlk olarak, Raspberry Pi (Kısaca Rpi) geliştirme kartına işletim programını kurmamız lazım. Kullanacağımız program spesifik olarak Ubuntu 20.04 gerektirdiğinden Ubuntu 20.04 kuracağız. Ancak bu sürümün, Rpi 4’e flaşlamamız için gerekli olan flaş dosyası yok. Bu yüzden Ubuntu 20.04 Server versiyonunu kurup, sonrasında terminal üzerinden Desktop versiyonunu kuracağız. Bunun için önce Rpi’nin kendi flaş programından Ubuntu 20.04 Server Version’u kurup SD kartı flaşlıyoruz. Sonrasında Rpi’mizi bir monitöre bağlayıp, bir de klavye bağlıyoruz ve sırasıyla aşağıdaki komutları giriyoruz [1]
 ```bash
sudo apt-get install ubuntu-desktop
```
Sadece tek bir komutla, Ubuntu Desktop’u kurabiliriz.
## ROS Kurulumu
Kullanacağımız SLAM programı ROS (Robot Operating System) temelli olduğu için öncelikle ROS kurulumunu tamamlamamız lazım.
İşin en sancılı kısmı burası. Eğer Rpi 3 kullanıyorsanız kurulumu saatler hatta günler, eğer Rpi 4 kullanıyorsanız birkaç saat sürecektir. Kurulumu yaparken ROS’un kendi sitesindeki yolu takip edeceğiz[1]. 
İlk olarak, işletim sistemimizin packages.ros.org sitesinden gelen yüklemeleri kabul etmesini sağlıyoruz.
 ```bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
 ```
Daha sonrasında, Curl kuruyoruz, Curl ile bağlantıdan dosyayı indiriyoruz ve bu dosyayı güvenilen anahtarlar arasına ekliyoruz.
 ```bash
•	sudo apt install curl
•	curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
 ```
Sistemimizde ROS’un bütün kaynaklarından yararlanacağımız için tamamını, yani “full” hâlini indiriyoruz.
 ```bash
sudo apt instal ros-noetic-desktop-full
 ```
Ros kullandığımız her terminalde kaynağı belirtmemiz gerekiyor. Bunun önüne geçmek için aşağıdaki komutu çalıştırmamız lazım ancak her zaman çalışmayabilir.
 ```bash
•	echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
•	source ~/.bashrc
 ```
Şu an ROS’u yükledik.Şimdi ise kurulum için gerekli olan uygulamaları ve paketleri kuracağız.
 ```bash
sudo apt install python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential
 ```
Son olarak ise ROS’un çalışması için gerekli olan ROSCORE’u yükleyeceğiz ve çalıştıracağız.
 ```bash
sudo apt install python3-rosdep
sudo rosdep init
rosdep update
 ```
Bu şekilde ROS’u yüklemeyi ve kurmayı bitirdik. Eğer nasıl çalıştığını daha detaylı okumak isterseniz sitesini ziyaret edebilirsiniz.
## Hector SLAM Kurulumu-1[2]
Lokalizasyon ve Haritalandırma, yani SLAM, için bir sürü uygulama var (Google Cartographer, Gmapping, Hector SLAM vs.). Aslında bu uygulamalar, ROS sayesinde erişilen lazer tarama sonuçlarına göre işlem yapan uygulamalar; yani ROS temelli uygulamalar. Eğer matematiğini anlarsak kodlayabiliriz, ancak bu bile en az 1-2 ayımızı alır. Biz, bu uygulamada Hector Slam kullandık.
Öncelikle, bütün dosyaların içinde barınabilmesi için bir dosya, yani ‘workspace’ oluşturmamız lazım. Genellikle bu dosya ‘catkin_ws’ olarak adlandırılır çünkü Catkin, bu projede kullanacağımız yazılım inşası programıdır (software build system). Yazılım inşası programları, kod derleyicileri gibi çalışır ancak kod derleyicileri gibi tek bir dosyayı derlemektense dosya sisteminin diğer bölümlerinde depolanan kodu da derlemeye yardımcı olur. 
‘catkin_ws’ dosyasını oluşturduktan sonra dosyanın içine ‘src’ dosyası oluşturuyoruz ve sırasıyla şu adımları takip ediyoruz.
Öncelikle terminali /catkin_ws/src kaynağına yönlendiriyoruz.
 ```bash
cd /catkin_ws/src
 ``` 
daha sonrasında Robopeak şirketinin RPLidar için, sahip olduğumuz lidar, yazmış olduğu kütüphaneyi klonluyoruz.
  ```bash
git clone https://github.com/robopeak/rplidar_ros.git
 ```
daha sonrasında tekrardan catkin_ws dosyasına çıkıyoruz
 ```bash
cd ..
 ```
Şimdi yazacağımız kodu proje içinde iki kere, her bir Git clone’dan sonra, yapacağız. Bu seferki da sürebilir uzun da sürebilir ancak bir sonraki epeyce uzun sürecek. Kütüphaneyi, en başta anlattığımız gibi ‘inşa’ edeceğiz. Bu işlemi yapmak için terminale aşağıdaki komutu giriyoruz ve bekliyoruz.
 ```bash
catkin_make
 ```
bu işlem bittikten sonra ilk aşamayı tamamladık demektir. Şimdi çalışıp çalışmadığını gözlemleyelim. Bunun için öncelikle ROS’un kendisini çalıştıralım.
 ```
roscore
 ```
Sonrasında yeni bir terminal açalım ve kaynağı belirtelim. Bu işlemi açacağımız her terminal için yapmamız lazım
 ```bash
source /opt/ros/noetic/setup.bash
 ```
az önce Github’tan klonladığımız RPLidar kütüphanesindeki anlık haritalandırma dosyasını çalıştıralım ve deneyelim.
 ```bash
roslaunch rplidar_ros rplidar.launch
 ```
şimdi ise çalıştırdığımız komutun çıktılarını görmek için yeni bir terminal oluşturalım ve aşağıdaki komutu girelim
 ```bash
rostopic echo /scan
 ```
çıktıları harita olarak görmek isterseniz de aşağıdaki komut satırını çalıştırın
 ```bash
roslaunch rplidar_ros view_rplidar.launch
 ```
eğer ekranımıza lidarımızın lazer algılama sonucunda oluşturduğu harita geldiyse kurulumumuz doğru bir şekilde tamamlanmış demektir. Eğer tamamlanmadıysa port ile alakalı sıkıntılarımız vardır. Aşağıdaki komut satırlarını açtığınız her terminalde yapacağınız işlemlerden önce çalıştırarak sorunu çözebilirsiniz.
 ```bash
chmod 666 /dev/ttyUSB0
 ```
## Hector SLAM Kurulumu-2[2]
Sırada asıl meseleye, yani Hector SLAM’in kendi kurulumuna geçeceğiz. Önceki terminallerin hepsini kapatıyoruz, eğer bir komut hâlen çalışmakta olduğundan kapatamıyorsak terminalde CTRL+C komutunu çalıştırarak durdurabilir ve kapatabilirsiniz.
Şidmi yeni bir terminal açıyoruz ve terminali /catkin_ws/src kaynağına yönlendiriyoruz
 ```bash
cd /catkin_ws/src
 ```
TU Darmstadt üniversitesi tarafından hazırlanan Hector SLAM programını klonluyoruz
 ```bash
git clone https://github.com/tu-darmstadt-ros-pkg/hector_slam.git
 ```
sonrasında tekrardan catkin_ws dosyasına çıkıyoruz
 ```bash
cd ..
 ```
şimdi ise tekrar ‘catkin make’ komudunu çalışıracağız. Bu işlem kullandığınız Rpi modeline bağlı olarak saatler, hatta gün bile sürebilir.
 ```bash
catkin make
 ```
Bu işlem tamamlandıktan sonra terminali kapatıyoruz. Hector SLAM kütüphanesindeki dosyaların bazılarının içeriğini değiştirmemiz lazım ki çalışabilsin. Sırasıyla aşağıdaki adımları uyguluyoruz.
catkin_ws/src/hector_slam/hector_mapping/launch klasörüne geliyoruz  ve mapping_default.launch dosyasının Text Editör ile açıyoruz. Daha sonrasında 5. ve 6. satırları aşağıdakiler ile değiştiriyoruz. Satır başındaki boşlukların değiştirmeden önceki ile aynı olduğuna dikkat edin
 ```bash	
  <arg name="base_frame" default="base_link "/>
	<arg name="odom_frame" default="base_link"/>
 ```
Sonrasında ise 54. satırı aşağıdaki ile değiştiriyoruz
 ```bash
<node pkg="tf" type="static_transform_publisher" name="map_nav_broadcaster" args="0 0 0 0 0 0 base_link laser 100"/>
 ```
Sonrasında bu dosyayı kapatıyor ve catkin_ws/src/hector_slam/hector_slam_launch /launch konumuna gidiyoruz. Burada tutorial.launch dosyasını açıyor ve 7. satırı aşağıdaki ile değiştiriyoruz.
 ```bash
<param name="/use_sim_time" value="false"/>
 ```
Bu değişikliklerin sebebini öğrenmek istiyorsanız buradaki[3] videoyu izleyebilirsiniz.
## Hector SLAM Çalıştırma[2]
Kurulumumuzu tamamladık, şimdi de çalıştıracağız. Birazdan yazacağım her bir komut satırı, rengine göre ayrı bir terminali temsil ediyor. Örneğin siyah olanlar bir terminali, mavi olanlar ise onlardan bağımsız bir terminali simgeliyor. Aynı renkte olan komut satırlarını ise algoritmik olarak, yani ilk yazılanı önce olacak şekilde ve sıralı çalıştıracağız. Her terminal açtığınızda yazılması gerekenleri yazmayı unutmayın.
Bir önceki terminalleri kapatıp yeni bir terminal açıyoruz ve aplikasyonların düzgün
çalışabilmesi için gereken roscore’u başlatmak için aşağıdaki komutu giriyoruz.
 ```bash
roscore
 ```
sonrasında yeni bir terminal açıyoruz ve haritalandırma için verileri alacağımız lazer algılamayı başlatıyoruz
 ```bash
roslaunch rplidar_ros rplidar.launch
 ```
lazer algılama verilerinin çıktısını görmek için yeni bir terminal açıyor ve aşağıdaki komutu çalıştırıyoruz.
 ```bash
rostopic echo /scan
 ```
şimdi ise Hector SLAM programını açıyoruz
 ```bash
roslaunch hector_slam_launch tutorial.launch
 ```
işlemimiz bu kadardı. Artık sağlıklı bir şekilde haritalandırma yapıp bu haritalandırmayı hafızada tutabiliyoruz.
Ancak unutmamalıyız ki, kaydettiğimiz haritayı yeniden kullanamıyoruz. Yani her seferinde yeniden haritalandırma yapmamız gerekiyor. Eğer haritayı kaydedip ona göre işlem yapmak istiyorsanız “gmapping” aplikasyonu daha yararlı olacaktır.

# Verilere Göre Python Kodunu Yazma
Python kodunu ilerleyen günlerde ekleyeceğim
Kaynaklar:
[1] https://wiki.ros.org/noetic/Installation/Ubuntu
[2] https://www.youtube.com/watch?v=h16BGFK_V9Q&t=39s
[3] https://youtu.be/Qrtz0a7HaQ4?t=370
