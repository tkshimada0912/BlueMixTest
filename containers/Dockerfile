FROM centos:centos7
RUN yum install -y openssh-server openssh-clients
RUN mkdir /root/.ssh
RUN echo "ssh-dss AAAAB3NzaC1kc3MAAACBAMjobCDKF9BnleUzIzMrA8LyTdyzolqCv4IC2pOakXKWjKtZ7jhzP7tFEtUlXQrMDh8VzJ4wMCTovw7BS+ZHRpQ84e3PiiOTbaD4WuY10C+5ykR7lWrez4RfrEZauTTDlcumbZxNBBroaloLVyJVaLUt8nFjshHzpazhu3kq+5YxAAAAFQCBD82M+2LUOLkg6+NO9MDuW5LzBQAAAIEAyFLgz7u8Jh5ZUwSu8byrvi5MFAnsSjCfyJiO/mFA2o3R03C5EfM+SgJgxN5ZMYGc0DRKmE2aogOfG/fQ4TELG2EgW+5JqWqjuQtPlwV28wXdjs7kB4dvkWcDciU6twR9T8UEvw0IH3pebKKDhcaadFaFJ2SZgTCER83Do8lOTFcAAACBALhCQf12aBPKqS8NLL56M5rVf+sd2GE3Hzzv3wkHPjrd92FazhusNnxqumwFhTtx4KLVEGYqjMHpj7UnvH/if9jhm0U8WkZ1DcSgX0LwIpoDR0bDKkPZAh/J6vpzhHFwhiHoSfVO7cDuEfUoDJ1qnilrULaqehjt7imvQ+ebtrcH shimada@shimadabu-no-MacBook-Pro.local" > /root/.ssh/authorized_keys
RUN chmod 700 /root/.ssh
RUN chmod 600 /root/.ssh/authorized_keys
RUN /usr/bin/ssh-keygen -q -t rsa -f /etc/ssh/ssh_host_rsa_key -N ''
EXPOSE 22
CMD /usr/sbin/sshd -D
