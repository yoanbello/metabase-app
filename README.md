# metabase-app

Proyecto de ejemplo: despliega **Metabase** (BI, imagen pública) sobre la pila `ops.forge` v2.1.0.

## Qué hace

- **PostgreSQL** en contenedor rootless (Quadlet) con red interna.
- **Metabase** (`metabase/metabase`) conectado a esa BD, publicado en `:3000`.
- **Caddy** como reverse proxy con TLS (80/443) hacia Metabase.
- Backups con systemd-timer + monitor Beszel.

## Requisitos

- VM **Rocky Linux 9.2 o 10** con acceso SSH root.
- cualquier distribución compatible con la colección `ops.forge`.

## Uso

```bash
# 1. Entorno Python + dependencias
python3 -m venv .venv && source .venv/bin/activate
pip install "ansible-core>=2.16"
ansible-galaxy collection install -r requirements.yml

# 2. Secretos (una vez)
echo "mi-vault-pass" > vault_pass && chmod 600 vault_pass
export ANSIBLE_VAULT_PASSWORD_FILE=./vault_pass
ansible-vault encrypt inventory/group_vars/testing/secrets.yml

# 3. Editar la IP y la llave SSH del usuario app en inventory/group_vars/testing/vars.yml

# 4. Bootstrap (una vez, como root)
ansible-playbook playbooks/01-bootstrap.yml -l testing

# 5. Reiniciar si SELinux cambió
ssh root@IP reboot

# 6. Provisionar y desplegar
ansible-playbook playbooks/02-provision.yml -l testing
ansible-playbook playbooks/03-app.yml -l testing
```

## Acceso

- Abrir `https://<tu-dominio>` (Metabase y Caddy).
- Con `domain` como `.local` (sin DNS público) Metabase no tendrá cert ACME; usarás HTTP directo a `:3000`.

## Variabes clave

| Variable | Qué hace |
|---|---|
| `server_ip` | IP del servidor |
| `domain` | Dominio público para TLS |
| `app_image` | Imagen de la app (Metabase) |
| `app_host_port` / `app_container_port` | Puerto publicado / interno |
| `caddy_upstream` / `caddy_upstream_port` | A dónde apunta Caddy |
| `app_env_vars` | Variables de entorno → `EnvironmentFile` |
| `postgres_app_db/user/password` | BD + usuario + password |