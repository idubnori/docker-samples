# escape=`

# Setup shared variables
ARG ERLANG_VERSION=21.0.1
ARG RABBITMQ_VERSION=3.7.7

# Use server core to support erlang install
FROM microsoft/windowsservercore as source

# Setup PowerShell as default Run Shell
SHELL ["PowerShell.exe", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue'; "]

# Environment Variables (ARGs needed to see outer scope ARGs)
ARG ERLANG_VERSION
ENV ERLANG_VERSION=$ERLANG_VERSION

# Install Erlang OTP
RUN Write-Host -Object 'Downloading Erlang OTP' ; `
    $erlangInstaller = Join-Path -Path $env:Temp -ChildPath 'otp_win64.exe' ; `
    Invoke-WebRequest -UseBasicParsing -Uri $('http://erlang.org/download/otp_win64_{0}.exe' -f $env:ERLANG_VERSION) -OutFile $erlangInstaller ; `
    Unblock-File -Path $erlangInstaller ; `
    Write-Host -Object 'Installing Erlang OTP' ; `
    Start-Process -NoNewWindow -Wait -FilePath $erlangInstaller -ArgumentList /S, /D=c:\erlang ; `
    Write-Host -Object 'Removing Erlang OTP Installer' ; `
    Remove-Item -Path $erlangInstaller ; `
    Write-Host -Object 'Done Installing Erlang OTP'



# Start from nano server
FROM microsoft/nanoserver

# Setup PowerShell as default Run Shell
SHELL ["PowerShell.exe", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue'; "]

# Environment Variables (ARGs needed to see outer scope ARGs)
ARG RABBITMQ_VERSION
ENV ERLANG_HOME=c:\erlang `
    RABBITMQ_HOME=c:\rabbitmq `
    RABBITMQ_BASE=c:\data `
    ERLANG_VERSION=$ERLANG_VERSION `
    RABBITMQ_VERSION=$RABBITMQ_VERSION

# setup persistent folders
VOLUME $RABBITMQ_BASE

# Copy erlang and c++ runtime from windows core image
COPY --from=source C:\erlang\ $ERLANG_HOME
COPY --from=source C:\windows\system32\mfc120.dll C:\windows\system32
COPY --from=source C:\windows\system32\mfc120chs.dll C:\windows\system32
COPY --from=source C:\windows\system32\mfc120cht.dll C:\windows\system32
COPY --from=source C:\windows\system32\mfc120deu.dll C:\windows\system32
COPY --from=source C:\windows\system32\mfc120enu.dll C:\windows\system32
COPY --from=source C:\windows\system32\mfc120esn.dll C:\windows\system32
COPY --from=source C:\windows\system32\mfc120fra.dll C:\windows\system32
COPY --from=source C:\windows\system32\mfc120ita.dll C:\windows\system32
COPY --from=source C:\windows\system32\mfc120jpn.dll C:\windows\system32
COPY --from=source C:\windows\system32\mfc120kor.dll C:\windows\system32
COPY --from=source C:\windows\system32\mfc120rus.dll C:\windows\system32
COPY --from=source C:\windows\system32\mfc120u.dll C:\windows\system32
COPY --from=source C:\windows\system32\mfcm120.dll C:\windows\system32
COPY --from=source C:\windows\system32\mfcm120u.dll C:\windows\system32
#msvcp120.dll & msvcr120.dll already exists on nanoserver
#COPY --from=source C:\windows\system32\msvcp120.dll C:\windows\system32
#COPY --from=source C:\windows\system32\msvcr120.dll C:\windows\system32
COPY --from=source C:\windows\system32\vcamp120.dll C:\windows\system32
COPY --from=source C:\windows\system32\vccorlib120.dll C:\windows\system32
COPY --from=source C:\windows\system32\vcomp120.dll C:\windows\system32

# Install Erlang OTP & RabbitMQ
RUN Write-Host -Object 'Setup Environment Variables' ; `
    $env:Path = $('{0}\sbin;{1};{2}' -f $env:RABBITMQ_HOME, $env:RABBITMQ_BASE, $env:Path) ; `
    [Environment]::SetEnvironmentVariable('Path', $env:Path) ; `
    Write-Host -Object $('Path: {0}' -f $env:Path) ; `
    Write-Host -Object 'Downloading RabbitMQ Zip' ; `
    $rabbitZip = Join-Path -Path $env:Temp -ChildPath 'rabbitmq.zip' ; `
    Invoke-WebRequest -UseBasicParsing -Uri $('https://bintray.com/rabbitmq/windows/download_file?file_path=rabbitmq-server-windows-{0}.zip' -f $env:RABBITMQ_VERSION) -OutFile $rabbitZip ; `
    Unblock-File -Path $rabbitZip ; `
    Write-Host -Object 'Extracting RabbitMQ' ; `
    Expand-Archive -Path $rabbitZip -DestinationPath c:\ ; `
    Rename-Item c:\rabbitmq_server-$env:RABBITMQ_VERSION $env:RABBITMQ_HOME ; `
    Write-Host -Object 'Removing RabbitMQ Zip' ; `
    Remove-Item -Path $rabbitZip ; `
    Write-Host -Object 'Done Installing RabbitMQ'

# setup working directory
WORKDIR $RABBITMQ_HOME\sbin

# Ports
# 4369: epmd, a peer discovery service used by RabbitMQ nodes and CLI tools
# 5672: used by AMQP 0-9-1 and 1.0 clients without TLS
# 5671: used by AMQP 0-9-1 and 1.0 clients with TLS
# 25672: used by Erlang distribution for inter-node and CLI tools communication and is allocated from a dynamic range (limited to a single port by default, computed as AMQP port + 20000).
# 15672: HTTP API clients and rabbitmqadmin (only if the management plugin is enabled)
# 61613: STOMP clients without TLS (only if the STOMP plugin is enabled)
# 61614: STOMP clients with TLS (only if the STOMP plugin is enabled)
# 1883: MQTT clients without TLS, if the MQTT plugin is enabled
# 8883: MQTT clients with TLS, if the MQTT plugin is enabled
# 15674: STOMP-over-WebSockets clients (only if the Web STOMP plugin is enabled)
# 15675: MQTT-over-WebSockets clients (only if the Web MQTT plugin is enabled)
EXPOSE 4369 5672 5671 25672 15672 61613 61614 1883 8883 15674 15675

# turn on management plugin
RUN .\rabbitmq-plugins.bat enable rabbitmq_management --offline

# run external command when container starts to allow for additional setup
CMD .\rabbitmq-server.bat
