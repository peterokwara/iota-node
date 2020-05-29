FROM ubuntu:18.04

RUN apt-get update 
RUN apt-get install -y git openjdk-8-jdk wget unzip
RUN wget https://github.com/bazelbuild/bazel/releases/download/0.28.1/bazel-0.28.1-installer-linux-x86_64.sh
RUN chmod +x bazel-0.28.1-installer-linux-x86_64.sh
RUN ./bazel-0.28.1-installer-linux-x86_64.sh 
RUN git clone https://github.com/iotaledger/compass.git
RUN apt-get install -y pkg-config zip g++ zlib1g-dev unzip python3
RUN apt-get install -y python
RUN cd compass && bazel run //docker:layers_calculator
