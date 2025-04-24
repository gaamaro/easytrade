.PHONY: deploy-all deploy check-health

deploy:
	@echo "🚀 Iniciando deploy na EC2 remota..."
	@for i in $$(seq 1 3); do \
		echo "Tentativa $$i de 3..."; \
		if ssh -o BatchMode=yes ec2 'cd /repo/easytrade && \
			export $$(cat dynatrace.env | xargs) && \
			docker compose pull && \
			docker compose up --force-recreate --remove-orphans --detach'; then \
			echo "✅ Deploy iniciado com sucesso!"; \
			break; \
		fi; \
		if [ "$$i" = "3" ]; then \
			echo "❌ Falha no deploy após 3 tentativas"; \
			exit 1; \
		fi; \
		echo "⏳ Aguardando 10 segundos antes da próxima tentativa..."; \
		sleep 10; \
	done

check-health:
	@echo "🔍 Verificando saúde dos containers na EC2..."
	@echo "⏳ Aguardando inicialização dos containers..."
	@for i in $$(seq 1 6); do \
		echo "Tentativa $$i de 6..."; \
		sleep 10; \
		if ssh -o BatchMode=yes ec2 'docker ps --format "{{.Names}} - {{.Status}}" | grep -i "up" | wc -l' > /dev/null; then \
			CONTAINERS_UP=$$(ssh -o BatchMode=yes ec2 'docker ps --format "{{.Names}} - {{.Status}}" | grep -i "up" | wc -l'); \
			CONTAINERS_TOTAL=$$(ssh -o BatchMode=yes ec2 'docker ps --format "{{.Names}}" | wc -l'); \
			if [ "$$CONTAINERS_UP" = "$$CONTAINERS_TOTAL" ]; then \
				echo "✅ Todos os containers estão em execução!"; \
				break; \
			fi; \
		fi; \
		if [ "$$i" = "6" ]; then \
			echo "❌ Timeout: Nem todos os containers iniciaram após 60 segundos"; \
			ssh -o BatchMode=yes ec2 'docker ps --format "{{.Names}} - {{.Status}}"'; \
			exit 1; \
		fi; \
	done
	@echo "🔍 Verificando status de saúde..."
	@if ssh -o BatchMode=yes ec2 'docker ps --format "{{.Names}} - {{.Status}}" | grep -i "unhealthy" | wc -l' > /dev/null; then \
		UNHEALTHY=$$(ssh -o BatchMode=yes ec2 'docker ps --format "{{.Names}} - {{.Status}}" | grep -i "unhealthy" | wc -l'); \
		if [ "$$UNHEALTHY" -gt 0 ]; then \
			echo "⚠️ Containers unhealthy encontrados:"; \
			ssh -o BatchMode=yes ec2 'docker ps --format "{{.Names}} - {{.Status}}" | grep -i "unhealthy"'; \
			exit 1; \
		else \
			echo "✅ Todos os containers estão saudáveis!"; \
		fi; \
	else \
		echo "❌ Erro ao verificar status dos containers"; \
		exit 1; \
	fi

deploy-all: deploy check-health
	@echo "✨ Deploy finalizado com sucesso!" 