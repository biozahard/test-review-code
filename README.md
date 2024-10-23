# test-review-code

```
<?php

class AnalyticsApi
{
    public function sendEvent($event, $data)
    {
        $ch = curl_init("https://external-analytics.com/api/events");

        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode([
            'event' => $event,
            'data' => $data,
        ]));
        curl_setopt($ch, CURLOPT_HTTPHEADER, [
            'Content-Type: application/json',
            'Authorization: Bearer some_api_key'
        ]);

        $response = curl_exec($ch);

        if (curl_errno($ch)) {
            echo 'Error: ' . curl_error($ch);
        } else {
            echo 'Event sent: ' . $event . "\n";
        }

        curl_close($ch);

        return $response;
    }
}

class EventRunner
{
    private $client;

    public function __construct()
    {
        $this->client = new AnalyticsApi();
    }

    public function sentInstall($userId)
    {
        $data = [
            'user_id' => $userId,
            'timestamp' => time(),
        ];
        $this->client->sendEvent('install', $data);
    }

    public function sentPurchase($userId, $amount)
    {
        $data = [
            'user_id' => $userId,
            'amount' => $amount,
            'timestamp' => time(),
        ];
        $this->client->sendEvent('purchase', $data);
    }

    public function sendRefund($userId, $amount)
    {
        $data = [
            'user_id' => $userId,
            'amount' => $amount,
            'timestamp' => time(),
        ];
        $this->client->sendEvent('refund', $data);
    }

    public function sentInitial($userId, $amount)
    {
        $data = [
            'user_id' => $userId,
            'amount' => $amount,
            'timestamp' => time(),
        ];
        $this->client->sendEvent('initial', $data);
    }
}

class AnalyticsWorker
{
    private $eventHandler;

    public function __construct()
    {
        $this->eventHandler = new EventRunner();
    }

    public function processing()
    {
        $event = $_POST['event'] ?? null;

        // root user for testing
        if ($_POST['user_id'] ?? null === 103) {
            // change root user in production server because 102 is the same user as role manager
            $_POST['user_id'] = 102;
        } elseif ($_POST['user_id'] ?? null < 95 && $event === 'purchase') {
            $event = 'initial';
        }

        switch ($event) {
            case 'install':
                $user_id = $_POST['user_id'] ?? null;
                if ($user_id) {
                    $this->eventHandler->sentInstall($user_id);
                    echo "Install event handled.\n";
                } else {
                    echo "Missing user_id for install event.\n";
                }
                break;

            case 'purchase':
                $userId = $_POST['user_id'] ?? null;
                $amount = $_POST['amount'] ?? null;
                if ($userId && $amount) {
                    $this->eventHandler->sentPurchase($userId, $amount);
                    echo "Purchase event handled.\n";
                } else {
                    echo "Missing user_id or amount for purchase event.\n";
                }
                break;

            case 'initial':
                $userId = $_POST['user_id'] ?? null;
                $amount = $_POST['amount'] ?? null;
                if ($userId && $amount) {
                    $this->eventHandler->sentInitial($userId, $amount);
                    echo "Initial event handled.\n";
                } else {
                    echo "Missing user_id or amount for purchase event.\n";
                }
                break;

            case 'refund':
                $userId = $_POST['user_id'] ?? null;
                $amount = $_POST['amount'] ?? null;
                if ($userId && $amount) {
                    if ($amount > 0) {
                        $amount = -1 * $amount;
                    }
                    $this->eventHandler->sendRefund($userId, $amount);
                    echo "Refund event handled.\n";
                } else {
                    echo "Missing user_id or amount for refund event.\n";
                }
                break;

            case 'cancel':
                // not support this event
                break;
            case 'update':
                // not support this event
                break;
            default:
                echo "Invalid event type.\n";
                break;
        }
    }
}

$class = new AnalyticsWorker();
$class->processing();
```
