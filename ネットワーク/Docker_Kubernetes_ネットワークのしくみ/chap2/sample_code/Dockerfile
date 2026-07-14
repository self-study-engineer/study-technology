FROM python:3.12.3-alpine
ADD app.py /work/ 
WORKDIR /work/
RUN pip install flask
RUN apk add curl
CMD ["python", "app.py"]