# Postgres for Omni-Chat-AI: pgvector + auto-created extra databases.
# Used by platforms that can't run an init-script bind mount (e.g. Render). The entrypoint runs
# everything in /docker-entrypoint-initdb.d on first boot, creating langfuse / omni_ai / litellm.
FROM pgvector/pgvector:pg16
COPY init-langfuse.sh /docker-entrypoint-initdb.d/10-extra-databases.sh
