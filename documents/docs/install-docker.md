# Cài đặt Docker

Docker là một ứng dụng giúp đơn giản hóa quá trình quản lý các tiến trình ứng dụng trong containers. Containers cho phép bạn chạy ứng dụng của mình trong các tiến trình cô lập về tài nguyên. Chúng tương tự như máy ảo, nhưng containers di động hơn, thân thiện với tài nguyên hơn và phụ thuộc nhiều hơn vào hệ điều hành máy chủ.

Để có giới thiệu chi tiết về các thành phần khác nhau của một container Docker, hãy xem The Docker Ecosystem: An Introduction to Common Components.

Trong hướng dẫn này, bạn sẽ cài đặt và sử dụng Docker Community Edition (CE) trên Ubuntu. Bạn sẽ cài đặt Docker, làm việc với containers và images, và đẩy một image lên Docker Repository.

Hướng dẫn này đã được xác thực trên Ubuntu 20.04, 22.04 và 24.04. Các lệnh và bước cài đặt hoạt động nhất quán trên các phiên bản này. Thiết lập repository sử dụng phát hiện phiên bản động để đảm bảo tương thích.

Nếu bạn muốn cách 1-click để triển khai ứng dụng Docker lên máy chủ trực tiếp, hãy xem DigitalOcean App Platform, tự động xử lý việc triển khai và mở rộng containers.

Điểm chính:
- Cài đặt Docker sử dụng repository chính thức của Docker để đảm bảo bạn nhận được phiên bản ổn định mới nhất với các bản cập nhật bảo mật, thay vì dựa vào repository mặc định của Ubuntu.
- Quá trình cài đặt không phụ thuộc phiên bản và tự động phát hiện phiên bản Ubuntu của bạn bằng $(lsb_release -cs) và $(dpkg --print-architecture) để tương thích trên Ubuntu 20.04, 22.04 và 24.04.
- Bạn có thể chạy lệnh docker mà không cần sudo bằng cách thêm người dùng của bạn vào nhóm docker, cải thiện hiệu quả workflow nhưng yêu cầu xem xét bảo mật cẩn thận.
- Docker images và containers là các khái niệm khác biệt: images là mẫu chỉ đọc, trong khi containers là các instance đang chạy được tạo từ những images đó.
- Các cài đặt Docker hiện đại bao gồm Docker Compose như một plugin, có thể truy cập qua docker compose (không có dấu gạch ngang) thay vì công cụ docker-compose độc lập.

---

## Điều kiện tiên quyết

Để làm theo hướng dẫn này, bạn sẽ cần:

Một máy chủ Ubuntu được thiết lập theo hướng dẫn thiết lập máy chủ Ubuntu ban đầu, bao gồm người dùng sudo không phải root và firewall.
Một tài khoản trên Docker Hub nếu bạn muốn tạo images riêng và đẩy chúng lên Docker Hub, như được hiển thị trong Bước 7 và 8.

---

## Bước 1 — Cài đặt Docker

Gói cài đặt Docker có sẵn trong repository chính thức Ubuntu có thể không phải là phiên bản mới nhất. Để đảm bảo chúng ta có phiên bản mới nhất, chúng ta sẽ cài đặt Docker từ repository chính thức của Docker. Để làm điều đó, chúng ta sẽ thêm một nguồn gói mới, thêm khóa GPG từ Docker để đảm bảo các tải xuống hợp lệ, và sau đó cài đặt gói.

Đầu tiên, cập nhật danh sách gói hiện tại của bạn:

```bash
sudo apt update
```

Tiếp theo, cài đặt một số gói tiên quyết cho phép apt sử dụng gói qua HTTPS:

```bash
sudo apt install ca-certificates curl gnupg
```

Sau đó thêm khóa GPG cho repository chính thức Docker vào hệ thống của bạn bằng phương pháp keyring hiện đại:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Thêm repository Docker vào APT sources. Lệnh này tự động phát hiện phiên bản Ubuntu của bạn:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Cập nhật cơ sở dữ liệu gói với các gói Docker từ repo mới được thêm:

```bash
sudo apt update
```

Xác minh rằng bạn sắp cài đặt từ repository Docker thay vì repository mặc định Ubuntu:

```bash
apt-cache policy docker-ce
```

Bạn sẽ thấy đầu ra cho biết Docker có sẵn từ repository Docker. Số phiên bản sẽ thay đổi, nhưng bạn nên thấy URL repository Docker trong đầu ra.

Cuối cùng, cài đặt Docker:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Lưu ý: Cài đặt bao gồm docker-buildx-plugin và docker-compose-plugin cung cấp chức năng Docker Compose hiện đại. Bạn có thể sử dụng docker compose (không có dấu gạch ngang) thay vì lệnh docker-compose độc lập cũ hơn.

Docker bây giờ sẽ được cài đặt, daemon được khởi động và quá trình được kích hoạt để khởi động khi boot. Kiểm tra xem nó có đang chạy không:

```bash
sudo systemctl status docker
```

Đầu ra sẽ tương tự như sau, cho thấy dịch vụ đang hoạt động và chạy:

```
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2020-05-19 17:00:41 UTC; 17s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 24321 (dockerd)
      Tasks: 8
     Memory: 46.4M
     CGroup: /system.slice/docker.service
             └─24321 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

Cài đặt Docker bây giờ cung cấp cho bạn không chỉ dịch vụ Docker (daemon) mà còn tiện ích dòng lệnh docker, hoặc Docker client. Chúng ta sẽ khám phá cách sử dụng lệnh docker sau trong hướng dẫn này.

---

## Bước 2 — Thực thi Lệnh Docker Không Cần Sudo (Tùy chọn)

Theo mặc định, lệnh docker chỉ có thể được chạy bởi người dùng root hoặc người dùng trong nhóm docker, được tạo tự động trong quá trình cài đặt Docker. Nếu bạn cố gắng chạy lệnh docker mà không thêm tiền tố sudo hoặc không ở trong nhóm docker, bạn sẽ thấy đầu ra như sau:

```
docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
```

Nếu bạn muốn tránh gõ sudo mỗi khi chạy lệnh docker, hãy thêm tên người dùng của bạn vào nhóm docker:

```bash
sudo usermod -aG docker ${USER}
```

Để áp dụng tư cách thành viên nhóm mới, hãy đăng xuất khỏi máy chủ và đăng nhập lại, hoặc gõ như sau:

```bash
su - ${USER}
```

Bạn sẽ được nhắc nhập mật khẩu người dùng của mình để tiếp tục.

Xác nhận rằng người dùng của bạn hiện đã được thêm vào nhóm docker bằng cách gõ:

```bash
groups
```

```
sammy sudo docker
```

Nếu bạn cần thêm người dùng vào nhóm docker mà bạn không đăng nhập, hãy khai báo tên người dùng một cách rõ ràng bằng:

```bash
sudo usermod -aG docker username
```

Phần còn lại của bài viết này giả định bạn đang chạy lệnh docker với tư cách người dùng trong nhóm docker. Nếu bạn chọn không, vui lòng thêm tiền tố cho các lệnh với sudo.

Hãy khám phá lệnh docker tiếp theo.

---

## Bước 3 — Sử dụng Lệnh Docker

Sử dụng docker bao gồm việc truyền cho nó một chuỗi các tùy chọn và lệnh theo sau là các đối số. Cú pháp lấy dạng này:

```bash
docker [option] [command] [arguments]
```

Để xem tất cả các subcommand có sẵn, gõ:

```bash
docker
```

Kể từ Docker 19, danh sách đầy đủ các subcommand có sẵn bao gồm:

```
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes
```

Để xem các tùy chọn có sẵn cho một lệnh cụ thể, gõ:

```bash
docker docker-subcommand --help
```

Để xem thông tin toàn hệ thống về Docker, sử dụng:

```bash
docker info
```

Hãy khám phá một số lệnh này. Chúng ta sẽ bắt đầu bằng cách làm việc với images.

---

## Bước 4 — Làm việc với Docker Images

Docker containers được xây dựng từ Docker images. Theo mặc định, Docker kéo các images này từ Docker Hub, một registry Docker được quản lý bởi Docker, công ty đứng sau dự án Docker. Bất kỳ ai cũng có thể lưu trữ Docker images của họ trên Docker Hub, vì vậy hầu hết các ứng dụng và bản phân phối Linux bạn cần sẽ có images được lưu trữ ở đó.

Để kiểm tra xem bạn có thể truy cập và tải xuống images từ Docker Hub không, gõ:

```bash
docker run hello-world
```

Đầu ra sẽ chỉ ra rằng Docker đang hoạt động chính xác:

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:6a65f928fb91fcfbc963f7aa6d57c8eeb426ad9a20c7ee045538ef34847f44f1
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

Docker ban đầu không thể tìm thấy image hello-world cục bộ, vì vậy nó tải image từ Docker Hub, là repository mặc định. Sau khi image tải xuống, Docker tạo một container từ image và ứng dụng trong container thực thi, hiển thị thông báo.

Bạn có thể tìm kiếm images có sẵn trên Docker Hub bằng cách sử dụng lệnh docker với subcommand search. Ví dụ, để tìm kiếm image Ubuntu, gõ:

```bash
docker search ubuntu
```

Script sẽ crawl Docker Hub và trả về danh sách tất cả images có tên khớp với chuỗi tìm kiếm. Trong trường hợp này, đầu ra sẽ tương tự như sau:

```
NAME                                                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
ubuntu                                                    Ubuntu is a Debian-based Linux operating sys…   10908               [OK]
dorowu/ubuntu-desktop-lxde-vnc                            Docker image to provide HTML5 VNC interface …   428                                     [OK]
rastasheep/ubuntu-sshd                                    Dockerized SSH service, built on top of offi…   244                                     [OK]
consol/ubuntu-xfce-vnc                                    Ubuntu container with "headless" VNC session…   218                                     [OK]
ubuntu-upstart                                            Upstart is an event-based replacement for th…   108                 [OK]
ansible/ubuntu14.04-ansible                               Ubuntu 14.04 LTS with
...
```

Trong cột OFFICIAL, OK cho biết một image được xây dựng và hỗ trợ bởi công ty đứng sau dự án. Khi bạn đã xác định image mà bạn muốn sử dụng, bạn có thể tải nó xuống máy tính của mình bằng subcommand pull.

Thực thi lệnh sau để tải image ubuntu chính thức xuống máy tính của bạn:

```bash
docker pull ubuntu
```

Bạn sẽ thấy đầu ra sau:

```
Using default tag: latest
latest: Pulling from library/ubuntu
d51af753c3d3: Pull complete
fc878cd0a91c: Pull complete
6154df8ff988: Pull complete
fee5db0ff82f: Pull complete
Digest: sha256:747d2dbbaaee995098c9792d99bd333c6783ce56150d1b11e333bbceed5c54d7
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

Sau khi một image đã được tải xuống, bạn có thể chạy một container bằng image đã tải xuống với subcommand run. Như bạn đã thấy với ví dụ hello-world, nếu một image chưa được tải xuống khi docker được thực thi với subcommand run, Docker client sẽ tải image trước, sau đó chạy một container bằng nó.

Để xem các images đã được tải xuống máy tính của bạn, gõ:

```bash
docker images
```

Đầu ra sẽ trông tương tự như sau:

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              1d622ef86b13        3 weeks ago         73.9MB
hello-world         latest              bf756fb1ae65        4 months ago        13.3kB
```

Như bạn sẽ thấy sau trong hướng dẫn này, images mà bạn sử dụng để chạy containers có thể được sửa đổi và sử dụng để tạo images mới, có thể được tải lên (pushed là thuật ngữ kỹ thuật) lên Docker Hub hoặc các registry Docker khác.

Hãy xem chi tiết cách chạy containers.

---

## Bước 5 — Chạy một Docker Container

Container hello-world mà bạn chạy ở bước trước là một ví dụ về container chạy và thoát sau khi phát ra thông điệp kiểm tra. Containers có thể hữu ích hơn nhiều, và chúng có thể tương tác. Sau tất cả, chúng tương tự như máy ảo, chỉ thân thiện với tài nguyên hơn.

Là một ví dụ, hãy chạy một container bằng image Ubuntu mới nhất. Sự kết hợp của các switch -i và -t cung cấp quyền truy cập shell tương tác vào container:

```bash
docker run -it ubuntu
```

Dòng lệnh của bạn sẽ thay đổi để phản ánh rằng bạn hiện đang làm việc bên trong container và sẽ có dạng này:

```
root@d9b100f2f636:/#
```

Lưu ý ID container trong dòng lệnh. Trong ví dụ này, nó là d9b100f2f636. Bạn sẽ cần ID container đó sau để xác định container khi bạn muốn xóa nó.

Bây giờ bạn có thể chạy bất kỳ lệnh nào bên trong container. Ví dụ, hãy cập nhật cơ sở dữ liệu gói bên trong container. Bạn không cần thêm tiền tố bất kỳ lệnh nào với sudo, vì bạn đang hoạt động bên trong container với tư cách người dùng root:

```bash
apt update
```

Sau đó cài đặt bất kỳ ứng dụng nào trong đó. Hãy cài đặt Node.js:

```bash
apt install nodejs
```

Điều này cài đặt Node.js trong container từ repository Ubuntu chính thức. Khi cài đặt hoàn tất, xác minh rằng Node.js đã được cài đặt:

```bash
node -v
```

Bạn sẽ thấy số phiên bản hiển thị trong terminal của bạn:

```
v10.19.0
```

Bất kỳ thay đổi nào bạn thực hiện bên trong container chỉ áp dụng cho container đó.

Để thoát khỏi container, gõ exit tại dấu nhắc.

Hãy xem quản lý containers trên hệ thống của chúng ta tiếp theo.

---

## Bước 6 — Quản lý Docker Containers

Sau khi sử dụng Docker một thời gian, bạn sẽ có nhiều container hoạt động (đang chạy) và không hoạt động trên máy tính của mình. Để xem các container hoạt động, sử dụng:

```bash
docker ps
```

Bạn sẽ thấy đầu ra tương tự như sau:

```
CONTAINER ID        IMAGE               COMMAND             CREATED
```

Trong hướng dẫn này, bạn đã khởi động hai containers; một từ image hello-world và một từ image ubuntu. Cả hai containers không còn chạy, nhưng chúng vẫn tồn tại trên hệ thống của bạn.

Để xem tất cả containers — hoạt động và không hoạt động, chạy docker ps với switch -a:

```bash
docker ps -a
```

Bạn sẽ thấy đầu ra tương tự như sau:

```
1c08a7a0d0e4        ubuntu              "/bin/bash"         2 minutes ago       Exited (0) 8 seconds ago                       quizzical_mcnulty
a707221a5f6c        hello-world         "/hello"            6 minutes ago       Exited (0) 6 minutes ago                       youthful_curie
```

Để xem container mới nhất bạn tạo, truyền nó switch -l:

```bash
docker ps -l
```

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
1c08a7a0d0e4        ubuntu              "/bin/bash"         2 minutes ago       Exited (0) 40 seconds ago                       quizzical_mcnulty
```

Để khởi động một container đã dừng, sử dụng docker start, theo sau là container ID hoặc tên container. Hãy khởi động container dựa trên Ubuntu với ID 1c08a7a0d0e4:

```bash
docker start 1c08a7a0d0e4
```

Container sẽ khởi động, và bạn có thể sử dụng docker ps để xem trạng thái của nó:

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
1c08a7a0d0e4        ubuntu              "/bin/bash"         3 minutes ago       Up 5 seconds                            quizzical_mcnulty
```

Để dừng một container đang chạy, sử dụng docker stop, theo sau là container ID hoặc tên. Lần này, chúng ta sẽ sử dụng tên mà Docker gán cho container, là quizzical_mcnulty:

```bash
docker stop quizzical_mcnulty
```

Khi bạn quyết định không còn cần một container nữa, xóa nó bằng lệnh docker rm, một lần nữa sử dụng container ID hoặc tên. Sử dụng lệnh docker ps -a để tìm container ID hoặc tên cho container liên quan đến image hello-world và xóa nó.

```bash
docker rm youthful_curie
```

Bạn có thể khởi động một container mới và đặt tên cho nó bằng switch --name. Bạn cũng có thể sử dụng switch --rm để tạo một container tự xóa khi nó dừng. Xem lệnh trợ giúp docker run để biết thêm thông tin về các tùy chọn này và những tùy chọn khác.

Containers có thể được biến thành images mà bạn có thể sử dụng để xây dựng containers mới. Hãy xem cách đó hoạt động.

---

## Bước 7 — Cam kết Thay đổi trong một Container thành một Docker Image

Khi bạn khởi động một Docker image, bạn có thể tạo, sửa đổi và xóa tệp giống như bạn có thể với máy ảo. Các thay đổi mà bạn thực hiện sẽ chỉ áp dụng cho container đó. Bạn có thể khởi động và dừng nó, nhưng một khi bạn phá hủy nó bằng lệnh docker rm, các thay đổi sẽ bị mất vĩnh viễn.

Phần này cho bạn thấy cách lưu trạng thái của một container như một Docker image mới.

Sau khi cài đặt Node.js bên trong container Ubuntu, bạn hiện có một container chạy từ một image, nhưng container khác với image bạn sử dụng để tạo nó. Nhưng bạn có thể muốn sử dụng lại container Node.js này làm cơ sở cho images mới sau này.

Sau đó cam kết các thay đổi thành một instance Docker image mới bằng lệnh sau.

```bash
docker commit -m "What you did to the image" -a "Author Name" container_id repository/new_image_name
```

Switch -m là cho thông điệp cam kết giúp bạn và những người khác biết những thay đổi bạn đã thực hiện, trong khi -a được sử dụng để chỉ định tác giả. Container_id là cái bạn lưu ý trước đó trong hướng dẫn khi bắt đầu phiên Docker tương tác. Trừ khi bạn tạo các repository bổ sung trên Docker Hub, repository thường là tên người dùng Docker Hub của bạn.

Ví dụ, cho người dùng sammy, với container ID d9b100f2f636, lệnh sẽ là:

```bash
docker commit -m "added Node.js" -a "sammy" d9b100f2f636 sammy/ubuntu-nodejs
```

Khi bạn cam kết một image, image mới được lưu cục bộ trên máy tính của bạn. Sau trong hướng dẫn này, bạn sẽ học cách đẩy một image lên một registry như Docker Hub để người khác có thể truy cập nó.

Liệt kê Docker images một lần nữa sẽ hiển thị image mới, cũng như cái cũ mà nó được lấy từ đó:

```bash
docker images
```

Bạn sẽ thấy đầu ra như sau:

```
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
sammy/ubuntu-nodejs   latest              7c1f35226ca6        7 seconds ago       179MB
...
```

Trong ví dụ này, ubuntu-nodejs là image mới, được lấy từ image ubuntu hiện có từ Docker Hub. Sự khác biệt về kích thước phản ánh các thay đổi đã được thực hiện. Và trong ví dụ này, thay đổi là NodeJS đã được cài đặt. Vì vậy, lần tới bạn cần chạy một container bằng Ubuntu với NodeJS được cài đặt sẵn, bạn chỉ cần sử dụng image mới.

Bạn cũng có thể xây dựng Images từ một Dockerfile, cho phép bạn tự động hóa cài đặt phần mềm trong một image mới. Tuy nhiên, điều đó nằm ngoài phạm vi của hướng dẫn này.

Bây giờ hãy chia sẻ image mới với người khác để họ có thể tạo containers từ nó.

---

## Bước 8 — Đẩy Docker Images lên một Docker Repository

Bước logic tiếp theo sau khi tạo một image mới từ một image hiện có là chia sẻ nó với một số bạn bè được chọn, thế giới trên Docker Hub, hoặc registry Docker khác mà bạn có quyền truy cập. Để đẩy một image lên Docker Hub hoặc registry Docker khác, bạn phải có tài khoản ở đó.

Phần này cho bạn thấy cách đẩy một Docker image lên Docker Hub. Để học cách tạo registry Docker riêng của bạn, hãy xem How To Set Up a Private Docker Registry on Ubuntu hoặc How To Set Up a Private Docker Registry on Ubuntu.

Để đẩy image của bạn, trước tiên đăng nhập vào Docker Hub.

```bash
docker login -u docker-registry-username
```

Bạn sẽ được nhắc xác thực bằng mật khẩu Docker Hub của mình. Nếu bạn chỉ định mật khẩu chính xác, xác thực sẽ thành công.

Lưu ý: Nếu tên người dùng registry Docker của bạn khác với tên người dùng cục bộ bạn sử dụng để tạo image, bạn sẽ phải gắn thẻ image của mình với tên người dùng registry của bạn. Cho ví dụ được đưa ra ở bước cuối cùng, bạn sẽ gõ:

```bash
docker tag sammy/ubuntu-nodejs docker-registry-username/ubuntu-nodejs
```

Sau đó bạn có thể đẩy image riêng của mình bằng:

```bash
docker push docker-registry-username/docker-image-name
```

Để đẩy image ubuntu-nodejs lên repository sammy, lệnh sẽ là:

```bash
docker push sammy/ubuntu-nodejs
```

Quá trình có thể mất một thời gian để hoàn thành khi nó tải lên các images, nhưng khi hoàn thành, đầu ra sẽ trông như sau:

```
The push refers to a repository [docker.io/sammy/ubuntu-nodejs]
e3fbbfb44187: Pushed
5f70bf18a086: Pushed
a3b5c80a4eba: Pushed
7f18b442972b: Pushed
3ce512daaf78: Pushed
7aae4540b42d: Pushed
...
```

Sau khi đẩy một image lên một registry, nó sẽ được liệt kê trên trang tổng quan tài khoản của bạn, như hiển thị trong hình ảnh dưới đây.

Image Docker mới liệt kê trên Docker Hub

Nếu một nỗ lực đẩy dẫn đến lỗi như sau, thì bạn có thể chưa đăng nhập:

```
The push refers to a repository [docker.io/sammy/ubuntu-nodejs]
e3fbbfb44187: Preparing
5f70bf18a086: Preparing
a3b5c80a4eba: Preparing
7f18b442972b: Preparing
3ce512daaf78: Preparing
7aae4540b42d: Waiting
unauthorized: authentication required
```

Đăng nhập với docker login và lặp lại nỗ lực đẩy. Sau đó xác minh rằng nó tồn tại trên trang repository Docker Hub của bạn.

Bây giờ bạn có thể sử dụng docker pull sammy/ubuntu-nodejs để kéo image lên một máy mới và sử dụng nó để chạy một container mới.

---

## Docker so với Docker Compose

Trong khi Docker cho phép bạn xây dựng và chạy containers, quản lý ứng dụng đa-container với chỉ Docker có thể tẻ nhạt. Đó là nơi Docker Compose xuất hiện.

Docker Compose là một công cụ để định nghĩa và chạy ứng dụng Docker đa-container bằng một tệp YAML. Thay vì chạy riêng biệt các containers cho máy chủ web, cơ sở dữ liệu và lớp caching, bạn có thể định nghĩa tất cả chúng trong một tệp docker-compose.yml và chạy chúng với:

```bash
docker-compose up
```

Để có hướng dẫn chi tiết về việc sử dụng Docker Compose, bao gồm cách thiết lập ứng dụng đa-container phức tạp và quản lý cấu hình của chúng, hãy xem hướng dẫn của chúng tôi về How To Install and Use Docker Compose on Ubuntu. Hướng dẫn này sẽ hướng dẫn bạn tạo và quản lý ứng dụng đa-container với Docker Compose, từ thiết lập cơ bản đến cấu hình phức tạp hơn.

Lưu ý: Các cài đặt Docker hiện đại bao gồm Docker Compose như một plugin. Sử dụng docker compose (không có dấu gạch ngang) thay vì lệnh docker-compose độc lập. Plugin cung cấp cùng chức năng với tích hợp tốt hơn.

| Tính năng | Docker CLI | Docker Compose |
|-----------|------------|----------------|
| Cách sử dụng | Hoạt động container đơn lẻ | Điều phối đa-container |
| Cấu hình | Lệnh CLI | Tệp cấu hình YAML |
| Xử lý phụ thuộc | Thủ công | Tự động xử lý dịch vụ liên kết |
| Trường hợp sử dụng tốt nhất | Kiểm tra containers cô lập | Phát triển cục bộ và thiết lập dàn |

Để biết thêm, xem How To Install and Use Docker Compose on Ubuntu.

---

## Khắc phục Sự cố Cài đặt Docker Phổ biến

**Vấn đề: docker: command not found**

**Khắc phục:** Docker CLI không có trong $PATH của bạn. Cài đặt lại Docker hoặc đảm bảo /usr/bin được bao gồm.

```bash
sudo apt install docker-ce docker-ce-cli containerd.io
```

**Vấn đề: Cannot connect to the Docker daemon**

**Khắc phục:** Docker không chạy, hoặc người dùng của bạn không ở trong nhóm docker.

```bash
sudo systemctl start docker
sudo usermod -aG docker $USER
```

Sau đó đăng xuất và đăng nhập lại.

**Vấn đề: Lỗi khóa GPG hoặc repository**

**Khắc phục:** Nếu bạn gặp lỗi khóa GPG hoặc repository, hãy tham khảo tài liệu cài đặt chính thức của Docker để có khóa và bước repository mới nhất. Phương pháp cài đặt hiện đại sử dụng /etc/apt/keyrings sẽ hoạt động trên Ubuntu 20.04, 22.04 và 24.04. Để cài đặt tự động có thể giúp tránh các vấn đề này, hãy xem hướng dẫn của chúng tôi về How To Use Ansible to Install and Set Up Docker on Ubuntu.

---

## Docker Desktop trên Ubuntu (Beta)

Docker Desktop hiện có sẵn ở phiên bản beta cho các bản phân phối Linux như Ubuntu. Nó cung cấp GUI, Docker Engine đi kèm và hỗ trợ Kubernetes.

Để cài đặt Docker Desktop trên Ubuntu:

```bash
sudo apt install ./docker-desktop-<version>-<arch>.deb
```

Tham khảo tài liệu Docker Desktop for Linux để biết các điều kiện tiên quyết và liên kết tải xuống.

Lưu ý: Docker Desktop phù hợp cho môi trường phát triển. Đối với cài đặt máy chủ, sử dụng Docker CE.

---

## Cài đặt Docker Sử dụng Dockerfile

Đối với môi trường tự động, cài đặt Docker bằng Dockerfile. Đây là ví dụ:

```dockerfile
FROM ubuntu:20.04

RUN apt-get update && \
    apt-get install -y ca-certificates curl gnupg && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg && \
    chmod a+r /etc/apt/keyrings/docker.gpg && \
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    apt-get update && \
    apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Đối với các tùy chọn tự động hóa nâng cao hơn, bạn cũng có thể sử dụng Ansible để cài đặt và cấu hình Docker. Xem hướng dẫn của chúng tôi về How To Use Ansible to Install and Set Up Docker on Ubuntu. Điều này đặc biệt hữu ích nếu bạn cần quản lý cài đặt Docker trên nhiều máy chủ hoặc muốn duy trì cấu hình nhất quán trong cơ sở hạ tầng của mình.

---

## Cách Gỡ Docker trên Ubuntu

Để xóa Docker khỏi hệ thống của bạn, chạy:

```bash
sudo apt purge docker-ce docker-ce-cli containerd.io
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
``` 