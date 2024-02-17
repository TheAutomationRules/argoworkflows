argoenv: installargo project_argoenv argocd_apps_argoenv

#----------------------------------------------------------------------------

installargo:
	@echo "| > INSTALACION DE ARGOCD"
	helm install argo-cd argo/argo-cd --version 5.51.6 \
      --namespace argocd \
      --create-namespace --wait \
      -f values-custom.yaml

project_argoenv:
	@echo "| > CREACION DE PROYECTO ARGOENV"
	kubectl create ns argoenv && kubectl apply -f argocd-manifests/project-argoenv.yaml

argocd_apps_argoenv:
	@echo "| > INSTALACION DE APLICACION ARGOCD APPS"
	kubectl apply -f argocd-manifests/application-argocd-apps-argoenv.yaml

argocdfwd:
	@echo "| > ARGOCD PORT FORWARD"
	kubectl port-forward service/argo-cd-argocd-server -n argocd 8080:443

argowffwd:
	@echo "| > ARGO WORKFLOWS PORT FORWARD"
	kubectl port-forward service/argo-workflows-server -n argo 2746:2746