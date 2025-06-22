

# Import Actual

<details><summary>Click to expand..</summary>



## Complementary Approach: Partial Mocking (Only if Necessary)

Use when only specific functions of a module need to be mocked and a full mock is overkill or impractical.

#### Example: `@google/genai`

```typescript
import * as googleGenAi from '@google/genai';
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { GoogleEmbeddingService } from './google-embedding-service';
import env from '../../src/env';

const mockProvider = vi.hoisted(() => {
    const mockEmbedContent = vi.fn();
    const mockGetGenerativeModel = vi.fn(() => ({}));

    const mockGoogleGenAI = vi.fn(() => ({
        getGenerativeModel: mockGetGenerativeModel,
        models: {
            embedContent: mockEmbedContent
        }
    }));

    return {
        mockEmbedContent,
        mockGetGenerativeModel,
        mockGoogleGenAI
    };
});

vi.mock('@google/genai', async () => {
    const actualPackage = await vi.importActual<typeof googleGenAi>('@google/genai');
    const { mockGoogleGenAI } = mockProvider;
    return {
        ...actualPackage,
        GoogleGenAI: mockGoogleGenAI
    };
});

describe('GoogleEmbeddingService()', () => {
    let service: GoogleEmbeddingService;

    beforeEach(() => {
        vi.stubEnv('GEMINI_API_KEY', env.GEMINI_API_KEY);
        service = new GoogleEmbeddingService();
    });

    describe('Constructor', () => {
        it('should initialize successfully with default values', () => {
            expect(service).toBeInstanceOf(GoogleEmbeddingService);
            expect(Reflect.get(service, '_apiKey')).toBe(env.GEMINI_API_KEY);
            expect(mockProvider.mockGoogleGenAI).toHaveBeenCalledWith(expect.objectContaining({
                apiKey: env.GEMINI_API_KEY
            }));
        });
    });
});
```






</details>