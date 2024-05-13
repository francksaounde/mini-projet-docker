FROM python:3.8-buster

LABEL maintainer="pozos-dsi"

WORKDIR /

COPY student_age.py .

RUN apt update -y && apt install python-dev python3-dev \
    libsasl2-dev python-dev libldap2-dev libssl-dev -y

COPY requirements.txt .

RUN pip3 install -r /requirements.txt

EXPOSE 5000

CMD ["python3", "./student_age.py"]

