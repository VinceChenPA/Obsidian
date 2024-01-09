Celery 是一个 基于python开发的分布式异步消息任务队列，通过它可以轻松的实现任务的异步处理， 如果你的业务场景中需要用到异步任务，就可以考虑使用celery

Celery 在执行任务时需要通过一个消息中间件（Broker）来接收和发送任务消息，以及存储任务结果， 一般使用rabbitMQ or Redis

![[v2-4211d9f0ddd4c971c26131b74274fa77_r.jpg]]