# forge-template

Template para proyectos que consumen `ops.forge` v2.0.0 (Podman rootless + Caddy + Quadlet).

**No editar forge directamente.** Este template es el punto de partida para cada proyecto.

## Arranque rápido

```bash
# 1. Clonar
git clone git@github.com:yoanbello/forge-template.git mi-proyecto
cd mi-proyecto

# 2. Entorno Python + dependencias
python3 -m venv .venv
source .venv/bin/activate
pip install "ansible-core>=2.16"
ansible-galaxy collection install -r requirements.yml

# 3. Secretos (una vez)
echo "mi-vault-pass" > vault_pass && chmod 600 vault_pass
export ANSIBLE_VAULT_PASSWORD_FILE=./vault_pass
ansible-vault encrypt inventory/group_vars/testing/secrets.yml

# 4. Editar IP dominio y llave en inventory/group_vars/testing/vars.yml

# 5. Bootstrap (una vez, como root)
ansible-playbook playbooks/01-bootstrap.yml -l testing

# 6. Reiniciar si SELinux cambió
ssh root@IP reboot

# 7. Provisionar y desplegar
ansible-playbook playbooks/02-provision.yml -l testing
ansible-playbook playbooks/03-app.yml -l testing
```

## Qué cambia entre proyecto y proyecto

Solo `inventory/group_vars/<env>/vars.yml` (IP, dominio, motor de BD) y `secrets.yml` (passwords).

## Estructura

```
ansible.cfg                  # config de Ansible
requirements.yml             # ops.forge v2.0.0 + deps
inventory/
  inventory.yml              # multi-entorno (prod, testing)
  group_vars/all.yml         # vars globales (runtime_user, firewall, etc.)
  group_vars/testing/vars.yml
  group_vars/testing/secrets.yml  # vault (no sube a git)
  group_vars/prod/vars.yml
  group_vars/prod/secrets.yml
playbooks/
  01-bootstrap.yml           # root → SELinux, firewalld, sshd, app, fail2ban
  02-provision.yml           # app  → podman, caddy, postgres, quadlet, backup, monitor
  03-app.yml                 # app  → despliegue de la aplicación
```

## Forge como dependencia

- `ops.forge` v2.0.0 se instala con `ansible-galaxy collection install -r requirements.yml`.
- Los roles se invocan con FQCN: `ops.forge.base`, `ops.forge.podman`, etc.
- Para actualizar forge: cambiar `version: 2.0.0` en `requirements.yml` y probar en un entorno de testing primero.
