FROM mcr.microsoft.com/dotnet/core/sdk:3.1-alpine

ENV NUGET_CLI_VERSION=v4.7.1 \
  PAKET_VERSION=5.226.0 \
  FAKE_VERSION=5.18.2

################################################################################
# For Dojo
################################################################################
ENV DOJO_VERSION=0.8.0
RUN apk update && apk upgrade && \
    apk add --no-cache bash git openssh wget curl sudo shadow nano xz
RUN git clone --depth 1 -b ${DOJO_VERSION} https://github.com/kudulab/dojo.git /tmp/dojo_git &&\
  /tmp/dojo_git/image_scripts/src/install.sh && \
  rm -r /tmp/dojo_git && \
  echo 'dojo ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers

################################################################################
# Above Dojo
################################################################################

RUN su dojo -c 'dotnet tool install fake-cli -g --version ${FAKE_VERSION}'
RUN su dojo -c 'dotnet tool install Paket -g --version ${PAKET_VERSION} --add-source https://api.nuget.org/v3/index.json'


RUN mkdir -p /home/dojo/.config/NuGet/ /home/dojo/.nuget/NuGet
COPY NuGet.Config /home/dojo/.nuget/NuGet/NuGet.Config
RUN ln -s /home/dojo/.nuget/NuGet/NuGet.Config /home/dojo/.config/NuGet/NuGet.Config &&\
  chown dojo:dojo -R /home/dojo/.nuget

COPY core/* /usr/bin/

# Warm up image
RUN su dojo -c 'mkdir -p /tmp/warmup && cd /tmp/warmup && dotnet new classlib && dotnet restore && dotnet build && rm -rf /tmp/warmup' &&\
 rm -rf /tmp/NuGetScratch
# Disable microsoft spying
ENV DOTNET_CLI_TELEMETRY_OPTOUT=1

# For Bash
COPY profile /home/dojo/.profile
COPY bashrc_default /home/dojo/.bashrc
RUN chown dojo:dojo -R /home/dojo

# Those are light to install and may change often, so they go at the bottom
# of this Dockerfile.
COPY etc_dojo.d/scripts/* /etc/dojo.d/scripts/

ENTRYPOINT ["/usr/bin/entrypoint.sh"]
CMD ["/bin/bash"]
