# Views

## Function-Based Views (FBVs)

Simple and explicit. Good for one-off endpoints:

```python
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response

@api_view(['GET', 'POST'])
@permission_classes([IsAuthenticated])
def worker_list(request):
    if request.method == 'GET':
        workers = Worker.objects.filter(company=request.user.company)
        serializer = WorkerSerializer(workers, many=True)
        return Response(serializer.data)

    serializer = WorkerSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    serializer.save(company=request.user.company)
    return Response(serializer.data, status=201)
```

## Class-Based APIViews

Better for grouping related HTTP methods:

```python
from rest_framework.views import APIView

class WorkerDetailView(APIView):
    permission_classes = [IsAuthenticated]

    def get_object(self, pk):
        return get_object_or_404(Worker, pk=pk, company=self.request.user.company)

    def get(self, request, pk):
        worker = self.get_object(pk)
        return Response(WorkerSerializer(worker).data)

    def patch(self, request, pk):
        worker = self.get_object(pk)
        serializer = WorkerSerializer(worker, data=request.data, partial=True)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data)

    def delete(self, request, pk):
        self.get_object(pk).delete()
        return Response(status=204)
```

## ViewSets

Best for full CRUD resources with routers:

```python
from rest_framework import viewsets, status
from rest_framework.decorators import action

class WorkerViewSet(viewsets.ModelViewSet):
    serializer_class = WorkerSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Worker.objects.filter(company=self.request.user.company)

    def perform_create(self, serializer):
        serializer.save(company=self.request.user.company)

    # Custom actions
    @action(detail=True, methods=['post'])
    def activate(self, request, pk=None):
        worker = self.get_object()
        worker.is_active = True
        worker.save(update_fields=['is_active'])
        return Response({'status': 'activated'})

    @action(detail=False, methods=['get'])
    def unassigned(self, request):
        workers = self.get_queryset().filter(crew__isnull=True)
        return Response(WorkerSerializer(workers, many=True).data)
```

## General best practices

- **Keep views thin.** Move business logic to models/services.
- **Use `get_object_or_404`.** Cleaner than `try/except`.
- **Use `serializer.is_valid(raise_exception=True)`.** Auto 400 response.
- **Filter by user/company in `get_queryset()`.** Security first.
- **Use `select_related` / `prefetch_related`.** Avoid N+1 queries.
- **Return proper status codes.** 201 create, 204 delete, 400 bad request.
- **Use `update_fields` in `.save()`.** Only update what changed.

## When to use what

| Scenario                  | Use                                        |
| ------------------------- | ------------------------------------------ |
| Full CRUD resource        | `ModelViewSet` + Router                    |
| List + Create only        | `mixins.ListModelMixin` + `GenericViewSet` |
| Custom logic, few methods | `APIView`                                  |
| One-off simple endpoint   | `@api_view` FBV                            |
