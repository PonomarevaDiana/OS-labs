FROM python:3.12-alpine AS builder

RUN apk add --no-cache gcc musl-dev python3-dev

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM python:3.12-alpine

RUN apk add --no-cache curl

ARG LAB_LOGIN
ARG LAB_TOKEN

LABEL org.lab.login=$LAB_LOGIN
LABEL org.lab.token=$LAB_TOKEN

RUN addgroup -S appuser && adduser -S appuser -G appuser

RUN mkdir -p /app && chown appuser:appuser /app

WORKDIR /app

COPY --from=builder /usr/local/lib/python3.12/site-packages/ /usr/local/lib/python3.12/site-packages/
COPY --from=builder /usr/local/bin/ /usr/local/bin/

COPY --chown=appuser:appuser app/ ./

USER appuser

ENV ROCKET_SIZE=Medium

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

CMD ["python", "app.py"]