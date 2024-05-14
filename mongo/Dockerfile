# before run stack file run this command
# docker build -t mongo:7.1 .
# mongosh --port 27017 -u root -p root --authenticationDatabase admin


FROM mongo:7.0
COPY mongo-keyfile /etc/mongo-keyfile
RUN chmod 400 /etc/mongo-keyfile
RUN chown mongodb:mongodb /etc/mongo-keyfile
CMD ["mongod"]
