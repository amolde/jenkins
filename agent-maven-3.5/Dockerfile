FROM openshift/jenkins-slave-base-centos7:v3.11

MAINTAINER OpenShift Developer Services <jenkins+openshift-dev-services@redhat.com>

ENV MAVEN_VERSION=3.5 \
    BASH_ENV=/usr/local/bin/scl_enable \
    ENV=/usr/local/bin/scl_enable \
    PROMPT_COMMAND=". /usr/local/bin/scl_enable" \
    MAVEN_OPTS="-Duser.home=$HOME"

# Install Maven
RUN INSTALL_PKGS="java-11-openjdk-devel java-1.8.0-openjdk-devel rh-maven35* unzip" && \
    x86_EXTRA_RPMS=$(if [ "$(uname -m)" == "x86_64" ]; then echo -n java-11-openjdk-devel.i686 java-1.8.0-openjdk-devel.i686; fi) && \
    yum install -y centos-release-scl-rh && \
    yum install -y --setopt=protected_multilib=false --enablerepo=centosplus $INSTALL_PKGS $x86_EXTRA_RPMS && \
    rpm -V java-11-openjdk-devel.x86_64 java-1.8.0-openjdk-devel.x86_64 rh-maven35 $x86_EXTRA_RPMS

RUN yum clean all -y && \
    mkdir -p $HOME/.m2

ENV JMETER_VERSION=5.3 \
    JMETER_PLUGIN_MGR_VERSION=1.4 \
    JMETER_CMD_RUNNER_VERSION=2.2

ENV JMETER_HOME="/opt/apache-jmeter-${JMETER_VERSION}" \
    JMETER_DOWNLOAD_URL=https://downloads.apache.org/jmeter/binaries/apache-jmeter-${JMETER_VERSION}.tgz \
    JMETER_PLUGIN_MGR_DWLD_URL="https://repo1.maven.org/maven2/kg/apc/jmeter-plugins-manager/${JMETER_PLUGIN_MGR_VERSION}/jmeter-plugins-manager-${JMETER_PLUGIN_MGR_VERSION}.jar" \
    JMETER_CMD_RUNNER_DWLD_URL="https://repo1.maven.org/maven2/kg/apc/cmdrunner/${JMETER_CMD_RUNNER_VERSION}/cmdrunner-${JMETER_CMD_RUNNER_VERSION}.jar"

ENV JMETER_BIN="${JMETER_HOME}/bin"

ENV PATH=${JMETER_BIN}:$PATH

RUN mkdir -p /tmp/dependencies && \
    curl "${JMETER_DOWNLOAD_URL}" -o /tmp/dependencies/apache-jmeter-${JMETER_VERSION}.tgz && mkdir -p /opt && tar -C /opt -xvf /tmp/dependencies/apache-jmeter-${JMETER_VERSION}.tgz && \
    curl -L --silent ${JMETER_PLUGIN_MGR_DWLD_URL} -o "${JMETER_HOME}/lib/ext/jmeter-plugins-manager.jar" && \
    curl -L --silent ${JMETER_CMD_RUNNER_DWLD_URL} -o "${JMETER_HOME}/lib/cmdrunner-${JMETER_CMD_RUNNER_VERSION}.jar" && \
    java -cp "${JMETER_HOME}/lib/ext/jmeter-plugins-manager.jar" org.jmeterplugins.repository.PluginManagerCMDInstaller && \
    rm -rf /tmp/dependencies

ENV JMETER_PLUGINS="jpgc-cmd,jpgc-json,jpgc-casutg"

RUN PluginsManagerCMD.sh install ${JMETER_PLUGINS}

RUN PluginsManagerCMD.sh status

# When bash is started non-interactively, to run a shell script, for example it
# looks for this variable and source the content of this file. This will enable
# the SCL for all scripts without need to do 'scl enable'.
ADD contrib/bin/* /usr/local/bin/
ADD ./contrib/settings.xml $HOME/.m2/

RUN chown -R 1001:0 $HOME && \
    chmod -R g+rw $HOME && \
    chown -R 1001:0 $JMETER_HOME && \
    chmod -R g+rw $JMETER_HOME

USER 1001
