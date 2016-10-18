FROM microsoft/windowsservercore
MAINTAINER contact@inetsoftware.de

# install chocolaty
RUN @powershell -NoProfile -ExecutionPolicy Bypass -Command "$env:chocolateyUseWindowsCompression = 'false' ; iwr https://chocolatey.org/install.ps1 -UseBasicParsing | iex"

# allow global confirmation for choco
RUN choco feature enable -n allowGlobalConfirmation

EXPOSE 22

#This monitor is from Microsoft 
ADD ServiceMonitor.exe /ServiceMonitor.exe

#install cygwin and cyg-get
RUN choco install cygwin cyg-get

# enable cygwin installations via cyg-get
ADD https://cygwin.com/setup-x86_64.exe cygwinsetup.exe
RUN @powershell -NoProfile -ExecutionPolicy Bypass -Command "cp /cygwinsetup.exe C:/tools/cygwin ; SETX Path \"C:\tools\cygwin\bin\;%PATH%\""

# install the ssh server package
RUN cyg-get openssh

# disable password policies
RUN @powershell -NoProfile -ExecutionPolicy Bypass -Command " \
    secedit /export /cfg c:\secpol.cfg ; \
    (gc C:\secpol.cfg).replace(\"PasswordComplexity = 1\", \"PasswordComplexity = 0\") | Out-File C:\secpol.cfg ; \
    secedit /configure /db c:\windows\security\local.sdb /cfg c:\secpol.cfg /areas SECURITYPOLICY ; \
    rm -force c:\secpol.cfg -confirm:$false \
    ; "

# set the administrator active
# create a jenkins user
# set a well know password
RUN @powershell -NoProfile -ExecutionPolicy Bypass -Command " \
    net user Administrator /active:yes ; \
    net user jenkins /add ; \
    net localgroup Administrators /add jenkins ; \
    Get-WmiObject win32_useraccount | Foreach-Object { \
    ([adsi](\"WinNT://\"+$_.caption).replace(\"\\\",\"/\")).SetPassword(\"jenkins\") } \
    ; "

# setup the sshd service 
RUN bash -c "ssh-host-config -y -u jenkins -w jenkins"

# link the previously created user account
RUN bash -c "mkpasswd -l -u jenkins > /etc/passwd && mkdir -p /home/jenkins"

# add the config for sshd
ADD sshd_config C:/sshd_config
RUN cp C:/sshd_config C:/tools/cygwin/etc/

# run endlessly
CMD ["C:\\ServiceMonitor.exe", "sshd"]