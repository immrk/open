from django.db import models
from django.contrib.auth.models import User

class Token(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    token = models.CharField(max_length=500, unique=True)
    created_at = models.DateTimeField(auto_now_add=True)
    is_valid = models.BooleanField(default=True)

    def __str__(self):
        return self.token

def is_token_valid(token):
    try:
        token_record = Token.objects.get(token=token)
        return token_record.is_valid
    except Token.DoesNotExist:
        return False

# decorators.py
from django.http import JsonResponse
from .utils import decode_jwt, is_token_valid

def jwt_required(view_func):
    def _wrapped_view(request, *args, **kwargs):
        token = request.META.get('HTTP_AUTHORIZATION')
        if token is None or not token.startswith('Bearer '):
            return JsonResponse({'error': 'Token missing'}, status=401)
        
        token = token.split(' ')[1]
        if not is_token_valid(token):
            return JsonResponse({'error': 'Invalid or expired token'}, status=401)
        
        payload = decode_jwt(token)
        if payload is None:
            return JsonResponse({'error': 'Invalid or expired token'}, status=401)
        
        request.user_id = payload['user_id']
        request.username = payload['username']
        return view_func(request, *args, **kwargs)
    
    return _wrapped_view

    token_record = Token.objects.get(token=token)
        token_record.is_valid = False
        token_record.save()
